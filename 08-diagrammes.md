# 8. Diagrammes de l’architecture

Les blocs Mermaid ci-dessous peuvent être copiés dans un visualiseur compatible Mermaid ou dans un README GitHub.

## 8.1 Architecture en couches initiale

```mermaid
flowchart TD
    Client[Client HTTP] --> C[Controller]
    C --> S[Service]
    S --> R[Repository concret]
    R --> DB[(Base de données)]
    S --> D[Modèle métier]
```

**Lecture :** le flux est simple, mais si `Service` dépend d’un repository concret ou si le modèle dépend de JPA, les détails techniques peuvent influencer le métier.

## 8.2 Architecture hexagonale cible

```mermaid
flowchart LR
    HTTP[Client HTTP] --> REST[Adaptateur entrant REST]
    CLI[CLI / tâche planifiée] --> IN[Port entrant : PasserCommandeUseCase]
    REST --> IN
    IN --> APP[Service applicatif]
    APP --> DOM[Domaine : Commande, LigneCommande]
    APP --> OUT[Port sortant : CommandeRepository]
    OUT -. implémenté par .-> MEM[Adaptateur mémoire]
    OUT -. implémenté par .-> JPA[Adaptateur JPA]
    MEM --> RAM[(Mémoire)]
    JPA --> DB[(Base SQL)]
```

**Important :** les flèches de flux d’exécution et les dépendances de code ne sont pas toujours la même chose. Le service appelle le port sortant, mais le code du port est défini dans le cœur. L’adaptateur dépend de cette abstraction pour l’implémenter.

## 8.3 Dépendances du code

```mermaid
flowchart BT
    DOMAIN[Domaine]
    APP[Application / cas d’usage]
    PORTS[Ports]
    ADAPTERS[Adaptateurs]
    APP --> DOMAIN
    APP --> PORTS
    ADAPTERS --> PORTS
    ADAPTERS --> APP
```

Ce schéma est volontairement simplifié. En pratique, l’adaptateur entrant dépend du port entrant ; l’adaptateur sortant dépend du port sortant. Évitez qu’un adaptateur d’infrastructure soit importé par le domaine.

## 8.4 Diagramme de séquence : créer une commande

```mermaid
sequenceDiagram
    actor Client
    participant REST as Adaptateur REST
    participant UC as Cas d’usage
    participant Dom as Domaine
    participant Port as Port Repository
    participant DB as Adaptateur de stockage

    Client->>REST: POST /api/commandes
    REST->>UC: executer(CommandeCommand)
    UC->>Dom: construire commande et vérifier invariants
    Dom-->>UC: Commande valide
    UC->>Port: enregistrer(commande)
    Port->>DB: écrire dans le stockage
    DB-->>Port: commande enregistrée
    Port-->>UC: commande
    UC-->>REST: résultat applicatif
    REST-->>Client: 201 Created + JSON
```

## 8.5 Structure de packages

```mermaid
flowchart TB
    subgraph core["Cœur"]
      D["domain"]
      A["application"]
      P["application.port.in / port.out"]
    end
    subgraph outside["Adaptateurs et assemblage"]
      W["adapter.in.web"]
      DB["adapter.out.persistence"]
      CFG["configuration"]
    end
    W --> P
    A --> D
    A --> P
    DB --> P
    CFG --> A
    CFG --> DB
```

## 8.6 Comment lire un diagramme hexagonal ?

Le dessin peut montrer un hexagone, un cercle ou des rectangles : la forme visuelle importe peu. Demandez plutôt :
1. Où se trouvent les règles métier ?
2. Qui possède les contrats ?
3. Les détails techniques peuvent-ils être remplacés ?
4. Peut-on tester les cas d’usage sans serveur ni base ?
5. Le domaine importe-t-il des packages techniques ?
