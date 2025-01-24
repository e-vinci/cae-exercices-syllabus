---
title: Fiche 2
description: Spring et basés de données.
permalink: posts/{{ title | slug }}/index.html
date: '2025-01-18'
tags: [fiche, spring]
---

# Spring et JPA

La semaine passée nous avons crée notre première application Spring, et appris différents concepts:

- Injection de dépendance
- Architecture en couche (Controller, Service)
- Mapping d'url à des méthodes
- Tests d'un API REST via le navigateur et les fichier ".http"

Cette semaine nous allons ajouter une brique supplémentaire - une couche "data" basée sur une base de données relationnelle - pour rappel la semaine passée nos données étaient de simple tableaux en Java (voir un fichier .json).

## Objectif du projet

Nous allons créer une API pour le point de vente d'un café. Celui-ci propose un ensemble de boissons. Il est donc nécessaire de pouvoir:

- Lister les boissons disponibles (une boisson a un nom, une description, un prix et le fait qu'elle soit alcolisée ou non)
- Recupérer les informations d'une boisson à partir de son identifiant
- Créer une nouvelle boisson
- Supprimer une boisson
- Editer une boisson
- Chercher parmis les boissons basé sur tout ou une partie du nom

Tous ces éléments doivent donc être crées sous forme de end points.

Ceci constitue une architecture "classique" de backend:

- Couche donnée
- Couche service (logique business)
- Couche web (request/response, etc)

A nouveau cette distinction peut paraître forcée dans l'exercice (on pourrait assez facilement mettre tout le code dans le controller) mais a du sens dès lors qu'on arrive sur des applications de taille réelle (plusieurs dizaines voir centaines de table, du code en dizaine de millier de lignes ou plus).

## Project setup

Nous allons créer un nouveau projet pour cette seconde fiche.

- New Project -> Spring Boot
    - Nom: fiche2
    - Language: Java
    - Type: Maven
    - Group: be.vinci.cae
    - Package: be.vinci.cae
    - JDK: Celui que vous avez installé
    - Java: 21
    - Packaging: Jar

Dans le second écran, sélectionnez:

- Lombok
- Spring Web
- Spring Data JPA
- H2 Database

Appuyez sur créer.

Nous avons donc deux dépendances supplémentaires:

- Spring Data JPA est un package de Spring avec tout un ensemble de classes et d'annotations pour facilier l'interaction entre le code Java et une base de donnée relationnelle
- H2 est une base de données relationnelle "in memory" (au sens qui tourne en mémoire). Elle fonctionne de manière identique à une base de donnée comme Postgresql, mais a l'avantage de pouvoir être installée beaucoup plus facilement. Il faut par contre bien retenir qu'elle est inadaptée à des applications en production (nous allons voir pourquoi).

La structure devrait vous être familière.

Ouvrez le fichier application.properties et ajoutez les éléments suivants:

```bash
spring.datasource.driver-class-name=org.h2.Driver
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect

spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.username=sa
spring.datasource.password=

spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.generate-ddl=true
```

Pour un peu de terminologie:
- JDBC (Java Database Connectivity) permet de connecter une application java à une source de donnée
- JPA - "Java Persistence API" est une *specification* sur la manière dont une application peut se connecter à une base de données. Ce standard permet que des développeurs comme nous puissons facilement utiliser différentes bases de données sans devoir tout réapprendre depuis le début. JPA fonctionne grâce à la couche JDBC
- Hibernate est une implémentation (la plus utilisée) de JPA

Petit schéma pour clarifier:

![](https://velog.velcdn.com/images/blaze241/post/2f6db67e-0918-42b3-8250-111f8224ec87/image.png)

JPA permettant de se connecter à quasiment n'importe quelle base de données nous devons spécifier un ensemble d'éléments.

- `driver-class-name` indique quelle classe utiliser pour intéragir avec la base de données (chaque base de donnée a son propre driver)
- Le dialecte est en hibernate (l'implémentation de JPA)

- `datasource-url` donne le "chemin" vers la base de donnée. Ici jdbc (la connection) : h2 (le type de base de données) : mem (en mémoire) : testdb (le nom de notre DB).
- On trouve après classiquement un user & password

Les deux derniers éléments indiquent à JPA de créer ou droper les tables et schéma en fonction de ce qui est défini dans le code (on va y revenir).

## Controller, Service, Model

Drink - as we did the day before

## Entities and Repositories

## Seed data and the ORM

## End to End List

## CRUD