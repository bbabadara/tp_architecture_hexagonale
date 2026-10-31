# 7. Tests et validation

## 7.1 La pyramide de tests appliquée au TP

```text
        /\
       /  \       Tests HTTP / intégration (peu nombreux)
      /----\
     /      \     Tests adaptateur de persistance
    /--------\
   /          \   Tests de cas d’usage
  /------------\
 /              \ Tests unitaires domaine (nombreux et rapides)
```

Ce dessin représente une stratégie, pas une loi mathématique. Les tests du domaine et des cas d’usage doivent être rapides et majoritaires ; les tests d’intégration vérifient que les composants réels sont bien assemblés.

## 7.2 Matrice de tests

| Zone | Ce qu’on teste | Dépendances |
|---|---|---|
| Domaine | invariants, calcul du total | JUnit |
| Cas d’usage | orchestration, appel du port | faux repository |
| Adaptateur REST | mapping DTO/HTTP, codes statut | Spring test / MockMvc |
| Persistance | mapping, lecture/écriture | base de test ou Testcontainers |
| Application complète | scénario bout en bout | contexte Spring + stockage |

## 7.3 Commandes

```bash
mvn test
mvn verify
```

- `test` compile et exécute les tests configurés.
- `verify` exécute les phases précédentes et les vérifications configurées dans Maven.

## 7.4 Contrôles manuels HTTP

Créer :

```http
POST /api/commandes
Content-Type: application/json
```

```json
{
  "clientId": "CLI-001",
  "lignes": [
    {"produit": "Clavier", "prixUnitaire": 12000, "quantite": 2},
    {"produit": "Souris", "prixUnitaire": 5000, "quantite": 1}
  ]
}
```

Vérifiez :
- statut `201` ;
- identifiant généré ;
- total égal à `29000` ;
- commande retrouvable ;
- une commande vide est rejetée avec un statut adapté ;
- prix négatif et quantité nulle sont rejetés.

## 7.5 Test d’architecture

Examinez les imports du package `domain`. Aucun import ne devrait venir de :
- `org.springframework...`
- `jakarta.persistence...`
- `jakarta.servlet...`
- `org.hibernate...`

Une règle ArchUnit peut automatiser ce contrôle. Exemple conceptuel : interdire aux classes du domaine de dépendre des packages `adapter`, `infrastructure` et `application` si le domaine doit rester indépendant. Adaptez la règle à votre structure réelle.

## 7.6 Critères de réussite

- les tests métier passent sans Spring ;
- les cas d’usage dépendent de ports ;
- les adaptateurs implémentent les ports ou appellent les ports entrants ;
- aucun contrôleur ne calcule le total ;
- le domaine ne connaît pas HTTP ni JPA ;
- remplacer l’adaptateur mémoire ne nécessite pas de modifier les règles métier.
