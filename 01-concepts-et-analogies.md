# 1. Concepts et analogies

## 1.1 Pourquoi changer d’architecture ?

Imaginez une boutique. Au départ, une seule application web gère les commandes. Le contrôleur reçoit la requête, vérifie les règles, enregistre en base et construit la réponse. Cela fonctionne, mais le code métier connaît peu à peu HTTP, SQL, le framework et les formats de données.

Le problème n’est pas que l’architecture en couches soit « mauvaise ». Elle devient fragile lorsque les dépendances techniques remontent vers les règles métier ou lorsque plusieurs couches se mélangent.

### Symptômes fréquents
- impossible de tester une règle métier sans démarrer Spring ou une base ;
- un changement de base de données oblige à modifier le service métier ;
- le contrôleur calcule lui-même le total d’une commande ;
- les entités JPA deviennent les objets métier par défaut ;
- les tests sont lents parce qu’ils passent tous par HTTP et la base.

## 1.2 Architecture en couches

Modèle courant :

- **Présentation** : HTTP, contrôleurs, DTO de requête/réponse.
- **Application / service** : orchestre un scénario.
- **Domaine** : règles et concepts métier.
- **Infrastructure** : base de données, clients externes, fichiers.

Dans une implémentation classique, les dépendances suivent souvent le sens du flux : contrôleur → service → repository → base. Le risque apparaît si le service dépend directement de détails techniques ou si le domaine importe des annotations JPA/Spring.

## 1.3 Analogie : un restaurant

Pensez à un restaurant :
- le client passe sa commande au serveur ;
- le serveur transmet la demande à la cuisine ;
- la cuisine applique les règles de préparation ;
- les ingrédients viennent d’un fournisseur.

La cuisine ne devrait pas avoir besoin de connaître le nom de l’application de livraison utilisée par le client. Elle doit pouvoir fonctionner avec différentes interfaces de commande et différents fournisseurs.

Dans notre logiciel :
- **domaine** = les règles de la cuisine ;
- **cas d’usage** = l’organisation d’une tâche (« passer une commande ») ;
- **port** = le contrat de collaboration ;
- **adaptateur** = le mécanisme concret qui parle HTTP, SQL, console ou à un système externe.

## 1.4 Ports et adaptateurs

Un **port** est une interface décrivant ce dont le cœur a besoin ou ce qu’il offre.
Un **adaptateur** relie ce contrat à une technologie concrète.

Deux grandes familles :

- **Port entrant (driving/inbound)** : permet à un acteur de déclencher un cas d’usage. Exemple : `PasserCommandeUseCase`.
- **Port sortant (driven/outbound)** : représente une dépendance dont le cœur/application a besoin. Exemple : `CommandeRepository`.

Adaptateurs entrants : contrôleur REST, interface CLI, tâche planifiée, consommateur Kafka.
Adaptateurs sortants : adaptateur JPA, client HTTP, système de fichiers, envoi d’e-mail.

## 1.5 Analogie : prise électrique

Une prise murale définit un contrat. Un appareil compatible peut s’y brancher sans que la prise ait besoin de connaître la marque de l’appareil. L’adaptateur de voyage permet de relier des standards différents.

Le port ne doit pas contenir le code SQL ou les détails HTTP. Il définit la capacité attendue. L’adaptateur traduit ce contrat vers la technologie.

## 1.6 Inversion des dépendances

Sans inversion :

```text
Service métier ───> JpaRepository concret ───> Base SQL
```

Avec inversion :

```text
Service applicatif ───> interface CommandeRepository
                              ▲
                              │ implémentée par
                    JpaCommandeRepository
```

Le code applicatif dépend d’une abstraction. L’infrastructure dépend aussi de cette abstraction et la réalise. Le flux d’exécution peut aller vers la base, tandis que le sens des dépendances du code reste dirigé vers le cœur.

## 1.7 Les zones du modèle hexagonal

- **Domaine** : entités métier, objets-valeurs, invariants, règles. Pas de Spring, HTTP, SQL ni dépendance vers les adaptateurs.
- **Application** : cas d’usage et orchestration. Elle appelle le domaine et les ports sortants.
- **Ports** : contrats entrants et sortants.
- **Adaptateurs** : traductions entre le monde extérieur et les ports.
- **Configuration** : assemble les composants concrets (injection de dépendances).

## 1.8 Ce que l’architecture hexagonale n’est pas

- Ce n’est pas « mettre chaque classe dans un package différent ».
- Ce n’est pas nécessairement du microservice.
- Ce n’est pas l’interdiction de Spring Boot ou de JPA.
- Ce n’est pas l’obligation d’avoir une interface pour chaque classe.
- Ce n’est pas une garantie automatique de qualité : les dépendances doivent réellement respecter les frontières.

## 1.9 Question de réflexion

Si demain on remplace une API REST par une commande console, quelles classes devraient changer ?

**Réponse attendue :** principalement l’adaptateur entrant. Le domaine et le cas d’usage devraient rester utilisables. Si le contrôleur contient les règles métier, la frontière est mal placée.
