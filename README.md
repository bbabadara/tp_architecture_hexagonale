# TP guidé — Migrer d’une architecture en couches vers l’architecture hexagonale

**Public :** développeurs débutants/intermédiaires en architecture logicielle  
**Technologie du fil rouge :** Java 21, Spring Boot 3, Maven, JUnit 5  
**Durée suggérée :** 1 journée (6 à 8 h), ou 2 séances de 3–4 h  
**Projet :** gestion de commandes d’une petite boutique

## Objectifs pédagogiques

À la fin du TP, vous saurez :
- reconnaître les dépendances typiques d’une architecture en couches ;
- distinguer domaine, application, ports et adaptateurs ;
- placer les règles métier sans les coupler à Spring, HTTP ou une base de données ;
- inverser les dépendances à l’aide d’interfaces (ports) ;
- tester les règles métier sans démarrer le serveur ni la base ;
- faire évoluer les adaptateurs sans réécrire le cœur métier.

## Parcours

1. `01-concepts-et-analogies.md` — concepts, analogies et vocabulaire
2. `02-enonce-et-regles-metier.md` — cahier des charges du TP
3. `03-etape-1-architecture-en-couches.md` — construire le point de départ
4. `04-etape-2-extraire-le-domaine.md` — isoler le modèle métier
5. `05-etape-3-ports-et-cas-usage.md` — ports, services applicatifs et inversion des dépendances
6. `06-etape-4-adaptateurs.md` — API REST et persistance
7. `07-tests-et-validation.md` — tests unitaires et tests d’intégration
8. `08-diagrammes.md` — diagrammes Mermaid et lecture guidée
9. `09-exercices-et-corrections.md` — exercices, indices et corrigés
10. `10-bonnes-pratiques-et-pieges.md` — checklist, pièges et critères d’évaluation
11. `annexes/structure-cible.md` — arborescence finale proposée

## Pré-requis

- Java 21 installé (`java -version`)
- Maven 3.9+ (`mvn -version`) ou wrapper Maven
- notions de classes, interfaces, exceptions, HTTP et tests unitaires
- IDE au choix (IntelliJ IDEA, VS Code, Eclipse)

> Ce pack est un support pédagogique. Les extraits de code sont progressifs : ils illustrent les décisions d’architecture et ne constituent pas, à eux seuls, un projet Spring Boot complet prêt à compiler. Suivez les étapes dans l’ordre et complétez les imports/configurations propres à votre projet.

## Comment travailler

Pour chaque étape :
1. lisez le problème avant de regarder la solution ;
2. dessinez les dépendances actuelles ;
3. codez une petite évolution ;
4. exécutez les tests ;
5. écrivez en deux phrases ce qui est devenu plus simple et ce qui reste couplé.

## Règle d’or

L’architecture hexagonale n’est pas une forme de dossier ni une obligation d’utiliser trois interfaces partout. C’est une manière de protéger le cœur métier et de faire dépendre les détails techniques de contrats définis par le cœur.
