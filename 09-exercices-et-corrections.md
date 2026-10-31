# 9. Exercices et corrections

## Exercice 1 — Classer les responsabilités

Classez chaque élément : domaine, application, port, adaptateur entrant, adaptateur sortant ou configuration.

1. Calcule le total des lignes.
2. Reçoit un `POST /api/commandes`.
3. Contrat `CommandeRepository`.
4. Appelle un repository et construit le résultat du scénario.
5. Convertit une commande en lignes SQL/JPA.
6. Relie l’implémentation JPA à l’interface du repository.

### Correction
1. Domaine.
2. Adaptateur entrant.
3. Port sortant.
4. Application / cas d’usage.
5. Adaptateur sortant.
6. Configuration / assemblage.

## Exercice 2 — Identifier la fuite de dépendance

```java
@Service
public class PasserCommandeService {
    private final CommandeJpaRepository repository;
}
```

Qu’est-ce qui pose problème ?

### Correction
Le cas d’usage dépend directement d’un composant de persistance concret. Il est plus difficile à tester sans infrastructure et le remplacement de JPA touche l’application. Introduire un port `CommandeRepository` dans le cœur et faire implémenter ce port par l’adaptateur JPA.

## Exercice 3 — Un contrôleur trop intelligent

```java
@PostMapping
ResponseEntity<?> creer(@RequestBody Request r) {
    BigDecimal total = r.lignes().stream()
        .map(l -> l.prix().multiply(BigDecimal.valueOf(l.quantite())))
        .reduce(BigDecimal.ZERO, BigDecimal::add);
    // sauvegarder...
}
```

Pourquoi est-ce une mauvaise frontière ?

### Correction
Le contrôleur contient une règle métier et la rend dépendante du transport HTTP. Une commande créée via CLI pourrait appliquer une autre formule. Le calcul doit être porté par le domaine ; le contrôleur se limite à traduire l’entrée et la sortie.

## Exercice 4 — Ajouter la recherche

Ajoutez le cas d’usage « consulter une commande par identifiant ».

### Indice
Créer un port entrant `ConsulterCommandeUseCase`, ou regrouper les cas d’usage dans une façade applicative si cela reste clair. Le cas d’usage appelle `CommandeRepository.trouverParId(id)`. La réponse HTTP `404` appartient à l’adaptateur REST.

### Correction indicative

```java
public interface ConsulterCommandeUseCase {
    Optional<CommandeResultat> executer(UUID id);
}
```

Le choix entre `Optional` et une exception applicative dépend de la convention retenue. Le contrôleur transforme l’absence en `404`.

## Exercice 5 — Ajouter un adaptateur CLI

Écrivez une commande console qui crée une commande en appelant `PasserCommandeUseCase`.

### Critère de réussite
Aucune modification du domaine ou du cas d’usage n’est nécessaire. Seul un nouvel adaptateur entrant est ajouté et configuré.

## Exercice 6 — Remplacer le stockage

Remplacez le repository mémoire par une implémentation JPA.

### Étapes attendues
1. conserver le port ;
2. créer les entités de persistance séparées si cela protège le domaine ;
3. écrire le mapping domaine ↔ persistance ;
4. implémenter le port ;
5. ajouter un test d’intégration de l’adaptateur ;
6. relancer les tests du domaine et du cas d’usage sans changement.

## Exercice 7 — Discussion d’architecture

Un projet CRUD de 4 écrans a-t-il toujours besoin d’une architecture hexagonale complète ?

### Réponse attendue
Pas nécessairement. Le niveau de séparation doit répondre aux risques, à la complexité métier, aux besoins de test et à la durée de vie du système. L’architecture hexagonale a un coût initial ; elle devient particulièrement utile quand les règles métier sont importantes, les interfaces multiples ou les dépendances techniques susceptibles d’évoluer.
