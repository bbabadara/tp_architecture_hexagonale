# 10. Bonnes pratiques, pièges et évaluation

## 10.1 Bonnes pratiques

- Garder les règles métier dans le domaine.
- Garder l’orchestration dans les cas d’usage.
- Nommer les ports avec le langage métier : `PasserCommandeUseCase`, `CommandeRepository`.
- Faire dépendre le cœur d’interfaces qu’il définit ou contrôle.
- Utiliser les DTO HTTP uniquement dans l’adaptateur web.
- Mapper les entités de persistance et le domaine lorsque cela apporte une vraie séparation.
- Injecter les dépendances à la composition, pas à l’intérieur des règles métier.
- Tester le domaine et les cas d’usage sans serveur.
- Introduire une abstraction quand elle protège une frontière ou facilite une substitution utile, pas par réflexe.
- Documenter le sens des dépendances, pas seulement l’arborescence des packages.

## 10.2 Pièges fréquents

### Piège 1 — Des dossiers propres, des dépendances sales
Mettre `domain/`, `application/`, `infrastructure/` ne suffit pas si le domaine importe `JpaRepository`.

### Piège 2 — Le domaine anémique
Si toutes les règles sont dans un service et que les entités ne font que transporter des données, les invariants peuvent être dispersés. Il est utile de placer les comportements qui protègent un concept métier dans ce concept.

### Piège 3 — Tout appeler « service »
Un service de domaine, un service applicatif et un service technique n’ont pas la même responsabilité. Choisissez des noms explicites.

### Piège 4 — Une interface pour chaque classe
Une interface n’est pas une fin en soi. Les ports représentent des frontières utiles ; ne créez pas des abstractions sans besoin.

### Piège 5 — Confondre port et adaptateur
`CommandeRepository` est le contrat. `CommandeJpaAdapter` est une implémentation concrète qui traduit le contrat vers JPA.

### Piège 6 — Faire fuiter les DTO
Ne passez pas `CommandeRequest` du contrôleur directement au domaine. Le format HTTP ne doit pas définir le modèle métier.

### Piège 7 — Sur-architecture
Pour une application très simple sans règles métier ni besoin de substitution, une séparation en couches peut être suffisante. L’architecture doit être proportionnée au problème.

## 10.3 Checklist finale

- [ ] Le domaine compile sans dépendance Spring/JPA.
- [ ] Les invariants sont vérifiés lors de la création des objets métier.
- [ ] Le cas d’usage orchestre et ne connaît pas HTTP.
- [ ] Les ports sont des contrats orientés métier.
- [ ] L’adaptateur REST traduit HTTP vers le port entrant.
- [ ] L’adaptateur de persistance implémente le port sortant.
- [ ] Le remplacement du stockage n’exige pas de modifier les règles métier.
- [ ] Les tests unitaires tournent sans infrastructure.
- [ ] Les erreurs sont converties en réponses HTTP appropriées.
- [ ] Les diagrammes correspondent au code réel.

## 10.4 Barème suggéré sur 20

| Critère | Points |
|---|---:|
| Domaine isolé et invariants corrects | 4 |
| Ports entrant/sortant correctement définis | 4 |
| Cas d’usage indépendant de Spring et de la persistance | 4 |
| Adaptateurs REST et stockage propres | 3 |
| Tests du domaine et du cas d’usage | 3 |
| Diagrammes, README et justification des choix | 2 |
| **Total** | **20** |

## 10.5 Questions de bilan

1. Quelle dépendance technique avez-vous réussi à retirer du cœur ?
2. Quel test est devenu plus facile ?
3. Que faudrait-il modifier pour ajouter une interface CLI ?
4. Quel coût supplémentaire l’architecture a-t-elle introduit ?
5. Dans quels projets ne choisiriez-vous pas cette architecture ?
