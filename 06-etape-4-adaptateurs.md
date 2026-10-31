# 6. Étape 4 — Construire les adaptateurs

## Objectif

Brancher l’application sur HTTP et sur un stockage, sans laisser ces détails pénétrer le domaine.

## 6.1 Adaptateur entrant REST

Le contrôleur traduit HTTP vers le modèle d’entrée du cas d’usage. Il ne porte pas les règles métier.

```java
@RestController
@RequestMapping("/api/commandes")
class CommandeRestAdapter {
    private final PasserCommandeUseCase passerCommande;

    CommandeRestAdapter(PasserCommandeUseCase passerCommande) {
        this.passerCommande = passerCommande;
    }

    @PostMapping
    ResponseEntity<CommandeResponse> creer(@RequestBody CreerCommandeRequest request) {
        var input = new CreerCommandeCommand(
                request.clientId(),
                request.lignes().stream()
                    .map(l -> new LigneCommandeCommand(
                            l.produit(), l.prixUnitaire(), l.quantite()))
                    .toList());
        var resultat = passerCommande.executer(input);
        return ResponseEntity.status(HttpStatus.CREATED)
                .body(CommandeResponse.depuis(resultat));
    }
}
```

Les DTO (`CreerCommandeRequest`, `CommandeResponse`) appartiennent à l’adaptateur web. Le cas d’usage ne doit pas les recevoir directement.

## 6.2 Adaptateur sortant en mémoire

Pour commencer, une implémentation mémoire est suffisante :

```java
@Repository
class CommandeMemoireAdapter implements CommandeRepository {
    private final Map<UUID, Commande> donnees = new ConcurrentHashMap<>();

    @Override
    public Commande enregistrer(Commande commande) {
        donnees.put(commande.id(), commande);
        return commande;
    }

    @Override
    public Optional<Commande> trouverParId(UUID id) {
        return Optional.ofNullable(donnees.get(id));
    }
}
```

Attention : `@Repository` est une annotation Spring. Elle est acceptable ici car l’adaptateur est une classe d’infrastructure. Pour un exercice de test pur, l’implémentation peut rester sans annotation.

## 6.3 Assemblage des dépendances

```java
@Configuration
class ConfigurationCommandes {
    @Bean
    Clock clock() {
        return Clock.systemUTC();
    }

    @Bean
    PasserCommandeUseCase passerCommandeUseCase(
            CommandeRepository repository, Clock clock) {
        return new PasserCommandeService(repository, clock);
    }
}
```

L’assemblage est un détail de démarrage. C’est ici que Spring relie les implémentations aux contrats.

## 6.4 Remplacer le stockage

Pour passer à JPA :
1. conserver l’interface `CommandeRepository` ;
2. créer `CommandeJpaAdapter implements CommandeRepository` ;
3. mapper le modèle de domaine vers des entités de persistance et inversement ;
4. conserver le même test du cas d’usage ;
5. ajouter des tests d’intégration spécifiques à l’adaptateur JPA.

Ne faites pas hériter le domaine de `JpaRepository` et n’ajoutez pas automatiquement `@Entity` au modèle de domaine juste pour gagner quelques lignes. Ce choix est possible dans certains compromis, mais il augmente le couplage entre modèle métier et persistance.

## 6.5 Gestion des erreurs HTTP

Créez un `@RestControllerAdvice` qui transforme les erreurs attendues en réponses HTTP cohérentes. Exemple de politique :
- validation métier → 400 ;
- commande inconnue → 404 ;
- erreur inattendue → 500 avec message générique et identifiant de corrélation si disponible.

Ne renvoyez jamais une stack trace ou des informations internes au client.

## 6.6 Défi facultatif

Ajoutez un adaptateur entrant CLI qui appelle le même `PasserCommandeUseCase`. Si vous devez modifier le domaine ou le cas d’usage pour permettre l’usage en console, examinez vos dépendances.
