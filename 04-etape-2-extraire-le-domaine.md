# 4. Étape 2 — Extraire le domaine

## Objectif

Créer un modèle métier qui reste testable sans Spring, HTTP ni base de données.

## 4.1 Objets métier

Une ligne de commande porte des données et des règles de validité. Une commande porte ses lignes et sait calculer son total.

Exemple simplifié :

```java
public record LigneCommande(String produit, BigDecimal prixUnitaire, int quantite) {
    public LigneCommande {
        if (produit == null || produit.isBlank()) {
            throw new IllegalArgumentException("Le produit est obligatoire");
        }
        if (prixUnitaire == null || prixUnitaire.signum() <= 0) {
            throw new IllegalArgumentException("Le prix doit être positif");
        }
        if (quantite < 1) {
            throw new IllegalArgumentException("La quantité doit être au moins 1");
        }
    }

    public BigDecimal sousTotal() {
        return prixUnitaire.multiply(BigDecimal.valueOf(quantite));
    }
}
```

Pourquoi `BigDecimal` plutôt que `double` ? Les nombres flottants binaires ne représentent pas exactement beaucoup de décimaux. Pour les montants monétaires, `BigDecimal` rend les opérations décimales plus prévisibles. Définissez aussi une politique d’échelle et d’arrondi si le métier en a besoin.

## 4.2 Entité Commande

```java
public record Commande(
        UUID id,
        String clientId,
        List<LigneCommande> lignes,
        Instant creeLe) {

    public Commande {
        if (id == null) throw new IllegalArgumentException("Identifiant obligatoire");
        if (clientId == null || clientId.isBlank()) {
            throw new IllegalArgumentException("Le client est obligatoire");
        }
        if (lignes == null || lignes.isEmpty()) {
            throw new IllegalArgumentException("La commande doit contenir une ligne");
        }
        lignes = List.copyOf(lignes);
        if (creeLe == null) throw new IllegalArgumentException("Date obligatoire");
    }

    public BigDecimal total() {
        return lignes.stream()
                .map(LigneCommande::sousTotal)
                .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
}
```

**Invariants** : règles qui doivent toujours rester vraies pour un objet valide. Ici, une commande ne peut pas exister sans client ni ligne.

Le constructeur protège l’objet : on ne peut pas créer facilement une commande invalide puis espérer qu’un contrôleur se souvienne de la vérifier.

## 4.3 Le domaine ne connaît pas le web

Évitez dans `domain` :
- `@RestController`, `@Service`, `@Entity` ;
- `HttpServletRequest`, `ResponseEntity` ;
- `JpaRepository`, `EntityManager` ;
- des DTO exclusivement liés au JSON.

Les annotations de validation ne sont pas automatiquement interdites partout, mais le but pédagogique est de rendre les invariants métier indépendants du framework.

## 4.4 Tests unitaires du domaine

```java
@Test
void calculeLeTotalDesLignes() {
    var clavier = new LigneCommande("Clavier", new BigDecimal("12000"), 2);
    var souris = new LigneCommande("Souris", new BigDecimal("5000"), 1);
    var commande = new Commande(
            UUID.randomUUID(), "CLI-001",
            List.of(clavier, souris), Instant.parse("2026-01-01T10:00:00Z"));

    assertEquals(new BigDecimal("29000"), commande.total());
}
```

Autres tests à écrire :
- une quantité nulle est refusée ;
- un prix nul ou négatif est refusé ;
- une liste de lignes vide est refusée ;
- le nom du produit vide est refusé ;
- la liste de lignes fournie ne peut pas être modifiée indirectement après création.

## 4.5 Point de contrôle

À cette étape, les tests du domaine doivent fonctionner avec JUnit seul, sans démarrer Spring ni la base. Si ce n’est pas le cas, cherchez une dépendance technique dans le domaine.
