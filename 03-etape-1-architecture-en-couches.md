# 3. Étape 1 — Construire puis observer l’architecture en couches

## Objectif

Partir d’une solution familière avant de la refactoriser. Ne cherchez pas encore la perfection : le but est de rendre visibles les couplages.

## 3.1 Arborescence de départ

```text
src/main/java/.../orders/
├── controller/CommandeController.java
├── service/CommandeService.java
├── repository/CommandeRepository.java
└── model/Commande.java
```

## 3.2 Contrat d’entrée HTTP

Le contrôleur doit :
1. recevoir un DTO JSON ;
2. appeler le service ;
3. traduire le résultat en réponse HTTP.

Il ne doit pas calculer le total ni appliquer les règles métier.

Exemple de forme :

```java
@RestController
@RequestMapping("/api/commandes")
class CommandeController {
    private final CommandeService service;

    CommandeController(CommandeService service) {
        this.service = service;
    }

    @PostMapping
    ResponseEntity<CommandeResponse> creer(
            @RequestBody CreerCommandeRequest request) {
        // Convertir le DTO, appeler le service, convertir la réponse.
        // Ne pas calculer le total ici.
        throw new UnsupportedOperationException("À compléter");
    }
}
```

## 3.3 Service initial

Dans la version en couches, le service risque de faire trop de choses :

```java
class CommandeService {
    private final CommandeRepository repository;

    CommandeService(CommandeRepository repository) {
        this.repository = repository;
    }

    Commande creer(CreerCommandeRequest request) {
        // 1. Valider client et lignes
        // 2. Calculer le total
        // 3. Construire la commande
        // 4. Enregistrer la commande
        // 5. Retourner la commande
        throw new UnsupportedOperationException("À compléter");
    }
}
```

Cette version peut être acceptable au départ, mais elle concentre validation, calcul et orchestration. L’exercice consiste à identifier ce qui relève du domaine et ce qui relève de l’application.

## 3.4 Repository en mémoire

```java
class CommandeRepository {
    private final Map<UUID, Commande> donnees = new HashMap<>();

    Commande enregistrer(Commande commande) {
        donnees.put(commande.id(), commande);
        return commande;
    }

    Optional<Commande> trouverParId(UUID id) {
        return Optional.ofNullable(donnees.get(id));
    }
}
```

Notez que cette classe est concrète. Si `CommandeService` la construit directement avec `new`, le couplage est encore plus fort. L’injection de dépendances améliore la substitution, mais une architecture hexagonale explicite le contrat et le sens des dépendances.

## 3.5 Cartographie des responsabilités

Pour chaque classe, répondez :

| Question | Exemple de réponse |
|---|---|
| Qui reçoit HTTP ? | Contrôleur |
| Qui décide si une commande est valide ? | À déplacer dans le domaine |
| Qui coordonne la création et l’enregistrement ? | Service applicatif |
| Qui connaît le stockage concret ? | Adaptateur de persistance |
| Qui calcule le total ? | Domaine |

## 3.6 Mini-défi

Modifiez la persistance en mémoire pour enregistrer aussi dans un fichier. Combien de classes faut-il modifier ? Si le service doit changer, notez pourquoi c’est un signe de frontière insuffisante.

**Livrable :** un schéma de l’architecture initiale et une liste de 3 couplages à réduire.
