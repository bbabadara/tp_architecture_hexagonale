# Annexe — Structure cible proposée

Exemple de packages Java. Adaptez le nom de base à votre projet.

```text
src/main/java/com/example/commandes/
├── domain/
│   ├── model/
│   │   ├── Commande.java
│   │   └── LigneCommande.java
│   └── exception/
├── application/
│   ├── port/
│   │   ├── in/
│   │   │   ├── PasserCommandeUseCase.java
│   │   │   └── ConsulterCommandeUseCase.java
│   │   └── out/
│   │       └── CommandeRepository.java
│   ├── command/
│   │   ├── CreerCommandeCommand.java
│   │   └── LigneCommandeCommand.java
│   ├── result/
│   │   └── CommandeResultat.java
│   └── service/
│       └── PasserCommandeService.java
├── adapter/
│   ├── in/
│   │   └── web/
│   │       ├── CommandeRestAdapter.java
│   │       ├── CreerCommandeRequest.java
│   │       └── CommandeResponse.java
│   └── out/
│       └── persistence/
│           ├── CommandeJpaAdapter.java
│           ├── CommandeJpaEntity.java
│           └── CommandeJpaMapper.java
└── configuration/
    └── CommandesConfiguration.java
```

## Règles de dépendance

```text
adapter.in.web ───────> application.port.in
application.service ──> application.port.out
application.service ──> domain
adapter.out.persistence ──> application.port.out
configuration ────────> assemble les implémentations
domain ───────────────> aucune couche technique
```

## Choix de conception à discuter

- Les ports peuvent être dans `application.port` ou proches du cas d’usage qui les utilise. Choisissez une convention cohérente.
- Les objets `Command` et `Result` ne sont pas des DTO HTTP : ils appartiennent à la frontière applicative.
- Un adaptateur JPA peut avoir ses propres entités et mapper vers le domaine.
- Pour une petite application, le mapping peut être manuel ; un mapper automatique n’est pas obligatoire.
- L’architecture hexagonale ne dicte pas une structure unique de packages. La règle essentielle est le contrôle des dépendances.
