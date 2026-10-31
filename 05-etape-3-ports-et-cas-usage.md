# 5. Étape 3 — Définir les ports et le cas d’usage

## Objectif

Faire dépendre l’application de contrats plutôt que d’une technologie de stockage.

## 5.1 Port entrant : ce que l’application offre

```java
public interface PasserCommandeUseCase {
    CommandeResultat executer(CreerCommandeCommand commande);
}
```

Le port entrant exprime une capacité métier. Le contrôleur REST appellera ce contrat. Un programme console pourrait l’appeler aussi.

`CreerCommandeCommand` est un objet de commande applicatif : il transporte les informations nécessaires au cas d’usage sans dépendre du format HTTP.

## 5.2 Port sortant : ce dont le cas d’usage a besoin

```java
public interface CommandeRepository {
    Commande enregistrer(Commande commande);
    Optional<Commande> trouverParId(UUID id);
}
```

Ce contrat appartient à la frontière du cœur applicatif. Il ne mentionne pas JPA, SQL ou Spring Data.

## 5.3 Implémentation du cas d’usage

```java
public final class PasserCommandeService implements PasserCommandeUseCase {
    private final CommandeRepository commandes;
    private final Clock clock;

    public PasserCommandeService(CommandeRepository commandes, Clock clock) {
        this.commandes = commandes;
        this.clock = clock;
    }

    @Override
    public CommandeResultat executer(CreerCommandeCommand input) {
        var lignes = input.lignes().stream()
                .map(l -> new LigneCommande(
                        l.produit(), l.prixUnitaire(), l.quantite()))
                .toList();

        var commande = new Commande(
                UUID.randomUUID(),
                input.clientId(),
                lignes,
                Instant.now(clock));

        var enregistree = commandes.enregistrer(commande);
        return CommandeResultat.depuis(enregistree);
    }
}
```

Le code illustre l’orchestration. `Clock` est injecté afin de rendre le temps contrôlable en test. L’identifiant pourrait aussi être fourni par un port `IdentifiantCommande`; ne créez pas une abstraction supplémentaire sans besoin réel.

## 5.4 Pourquoi le cas d’usage n’est-il pas le domaine ?

Le domaine décide des invariants et des calculs métier. Le cas d’usage coordonne les étapes : convertir l’entrée, construire l’objet valide, demander son enregistrement, présenter le résultat applicatif.

Analogie : le domaine décrit comment une commande doit être valide ; le cas d’usage est le responsable qui orchestre le processus de prise de commande.

## 5.5 Test du cas d’usage sans base

Créez un faux repository en mémoire dans le test :

```java
final class CommandeRepositoryMemoire implements CommandeRepository {
    private final Map<UUID, Commande> stockage = new HashMap<>();

    @Override
    public Commande enregistrer(Commande commande) {
        stockage.put(commande.id(), commande);
        return commande;
    }

    @Override
    public Optional<Commande> trouverParId(UUID id) {
        return Optional.ofNullable(stockage.get(id));
    }
}
```

Puis testez :
- le cas valide retourne une commande avec le bon total ;
- le repository reçoit la commande ;
- les règles du domaine empêchent une entrée invalide ;
- l’horloge fixe donne une date déterministe.

## 5.6 Sens des dépendances

`PasserCommandeService` importe `CommandeRepository`, une interface. Il n’importe pas `JpaCommandeRepository`. L’adaptateur d’infrastructure sera celui qui implémente le port.

**Livrable :** un test du cas d’usage qui passe sans Spring, sans serveur HTTP et sans base de données.
