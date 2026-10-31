# 2. Énoncé du TP — Gestion de commandes

## Contexte

Une boutique veut une API simple pour enregistrer des commandes. Dans un premier temps, les données peuvent être stockées en mémoire. Ensuite, nous introduirons un adaptateur de persistance.

## Fonctionnalités

1. Créer une commande contenant un identifiant client et une ou plusieurs lignes.
2. Chaque ligne contient un nom de produit, un prix unitaire et une quantité.
3. Calculer le total.
4. Refuser une commande sans ligne.
5. Refuser un prix inférieur ou égal à zéro.
6. Refuser une quantité inférieure à 1.
7. Retrouver une commande par identifiant.
8. Exposer l’action de création via HTTP.

## Règles métier

- Le client doit être renseigné.
- Une commande contient au moins une ligne.
- Le nom du produit n’est pas vide.
- Le prix unitaire est strictement positif.
- La quantité est strictement positive.
- Le total est calculé à partir des lignes ; il n’est pas fourni par le client.
- Une commande créée reçoit un identifiant.
- Une commande ne doit pas dépendre de Spring MVC, de JPA ni du format JSON.

## Travail demandé

**Partie A — état initial en couches**
1. Créez un contrôleur REST.
2. Créez un service qui valide les données et calcule le total.
3. Créez un repository concret en mémoire.
4. Repérez les responsabilités qui se chevauchent et les dépendances.

**Partie B — refactorisation hexagonale**
1. Déplacez les règles et le modèle vers le domaine.
2. Définissez un port entrant pour le cas d’usage.
3. Définissez un port sortant pour enregistrer et retrouver les commandes.
4. Implémentez le cas d’usage sans dépendance technique.
5. Transformez le contrôleur et le repository en adaptateurs.
6. Ajoutez des tests unitaires du domaine et du cas d’usage.

## Exemples d’entrée

```json
{
  "clientId": "CLI-001",
  "lignes": [
    {"produit": "Clavier", "prixUnitaire": 12000, "quantite": 2},
    {"produit": "Souris", "prixUnitaire": 5000, "quantite": 1}
  ]
}
```

Total attendu : `29000`.

## Résultats HTTP suggérés

- `201 Created` : commande créée.
- `400 Bad Request` : données invalides.
- `404 Not Found` : commande absente.
- `500 Internal Server Error` : erreur inattendue (ne pas exposer la trace technique au client).

## Limites pédagogiques

Pour garder le TP focalisé sur l’architecture, l’authentification, les paiements, la gestion de stock, les transactions distribuées et les taxes ne sont pas traités.
