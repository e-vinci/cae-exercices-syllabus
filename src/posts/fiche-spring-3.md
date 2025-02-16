---
title: Fiche 3
description: Spring et sécurité.
permalink: posts/{{ title | slug }}/index.html
date: '2025-01-30'
tags: [fiche, spring]
---

# Spring et JPA

La semaine passée nous avons appris les concepts suivants:

- Persistence des données
- Object-Relationnal Mapping

Cette semaine nous allons ajouter les derniers éléments important pour tout backend : la gestion de la sécurité et de l'authentification.

## Objectif du projet

Nous allons partir d'une API pour la gestion d'une pizzeria, dont les fonctionnalités suivantes ont été déjà développées :

- Lister les pizzas disponibles (une pizza a un nom et un contenu)
- Recupérer les informations d'une pizza à partir de son identifiant
- Créer une nouvelle pizza
- Supprimer une pizza
- Editer une pizza
- Commander une pizza
- Inscrire des utilisateurs (un utilisateur a un pseudonyme, un mot de passe en clair pour le moment et un rôle utilisateur par défaut)

L'objectif de ce tutoriel va être de protéger les actions qui doivent l'être. 

- Seuls les utilisateurs connectés pourront commander une pizza
- Seuls les administrateurs pourront créer, supprimer et éditer des pizzas.

On va donc couvrir ici deux notions différentes et liées:

- On parle d'`Authentification` pour tout ce qui est "est ce que la personne est bien qui elle prétend être". La manière la plus classique de gérer de l'authentification est via un user (souvent un email) et un password.
- On parle d'`Authorization` pour tout ce qui est "est ce que cette personne a le droit de faire ce qu'elle souhaite" - un user peut être correctement authentifié mais ne pas avoir les droits de faire une certaine opération (créer une nouvelle pizza dans notre exemple). La solution classique s'appelle des `Roles` ou [RBAC](https://en.wikipedia.org/wiki/Role-based_access_control) - un système par lequel chaque user se voit attribuer un ou plusieurs rôles qui lui donne des droits spécifique (user, administrateur ou par exemple pour un site de contenu: reader, editor, reviewer, ...)

On voit directement qu'on ne peut pas `authoriser` quelqu'un avant de l'avoir `authentifié`.

## Project setup

Nous allons cloner un repository contenant le code déjà existent pour le modifier dans cette troisième fiche.

Cloner le repository présent ici: [https://github.com/e-vinci/cae_exercices_fiche3](https://github.com/e-vinci/cae_exercices_fiche3).

Lancez le projet (avec le docker pour Postgres ou non selon vos préférence) et vérifiez que tout est en ordre.
Notez que le postgres fourni est fait pour tourner sur le port 5433 (et non sur le 5432 par défaut) dans le but de ne pas créer de conflit avec un Postgres installé sur votre machine.

### DTOs

Vous allez trouvez deux répertoire au niveau du package "models":
- Un "entities" qui correspond aux objets liés à la DB (les "Entités") comme précédemment
- Un nommé "DTOs" avec de "simples objets java" ("POJO" en anglais: "Plain Old Java Object")

Ceci suit un pattern architectural appellé ["Data Transfer Object"](https://martinfowler.com/eaaCatalog/dataTransferObject.html) - le site référencé est celui de Martin Fowler, un programmeur et architecte assez connu et prolifique notemment dans sa documentation de "patterns" - des éléments de code que l'on retrouve dans beaucoup d'applications. 

L'idée ici est d'éviter que les objet de type "Entity" ne soient utilisé en input du controller - il n'y a par exemple aucun sens que l'utilisateur fournissent l'id de l'objet, ou le password encrypté, bien que ces champs doivent exister (et être obligatoires) au niveau de l'entité User. On crée donc des objets distincts, très simples (des champs et des accesseurs) pour ce faire.

## Exercice

Le tour du projet étant fait et fonctionnel, nous allons ajouter ces features de sécurité.

### Nouvelles dépendances

Nous allons ajouter des dépendances nécessaires:

- Dans le pom.xml, !["edit starters"](https://cae-exercices.e-vinci.be/images/starters.png) et ajouter spring security (Edit Starter permet de compléter vos settings après la création du projet)
- Editer le pom.xml pour ajouter java-jwt et spring-dotenv - pour ce faire, utilisez "Generate... > Maven Dependency"

Faite attention de prendre les packages exacts spécifiés (il y en a beaucoup, les confusions sont faciles !).

Pourquoi ces packages:

- Spring Security va nous permettre de facilement configurer les accès à nos différent endpoints
- JWT (JSON Web Token) est une technologie pour générer des "token" après une authentification réussie. Les token servent d'alternatives aux cookies dès lors que l'on utilise un client type SPA (comme React) - des éléments sur ce sujet [ici](https://stackoverflow.com/questions/37582444/jwt-vs-cookies-for-token-based-authentication)
- Spring Dotenv va nous aider à garder nos secrets (typiquement user/password utilisé pour la database) dans un format plus facile à manipuler que des variables d'environnement.

## Hachage des mots de passe

Utiliser bcrypt (inclus avec spring security) pour hacher le mdp dans le service avant de l'enregistrer dans la DB.
Créer le fichier de configuration définissant le bean permettant d'avoir un BCryptPasswordEncoder.

## Création de token JWT

Ajouter une route login qui récupère les credentials. Le service vérifie que l'utilisateur existe et que son mot de passe est correct (avec bcrypt), puis génère un token jwt avec java-jwt. Inclure le pseudonyme dans le token, rappeler le fonctionnement de jwt. Le secret est hardcodé pour le moment.

## Configuration de Spring Security

Créer le fichier filtre, qui vérifie le token jwt avec java-jwt. Le secret y est encore hardcodé.
Créer le fichier de configuration de Spring Security, qui définit method security et l'utilisation du filtre.
Ajouter les annotations de sécurité sur les routes qui en ont besoin :
- Créer, supprimer et éditer une pizza pour les admins
- Commander une pizza pour les utilisateurs connectés

## Sécurisation de properties

Créer un fichier .env définissant des variables d'environnement.
Modifier le fichier application.properties pour utiliser ces variables d'environnement.
Créer classe de configuration pour récupérer la propriété du secret.
Utiliser la config pour le secret au lieu de la valeur hardcodée.

# Documentation

## Documentation OpenAPI

Description de ce qu'est OpenAPI et Swagger.
Différences de format. Nous utilisons OpenAPI 3.X en YAML.

## Création et visualisation

Editeur en ligne, linter et prévisualisation.
Création dans IntelliJ ("New" -> "Documentation"), onglet de prévisualisation.

## Syntaxe

- Infos générales
- Schémas
- Chemins
- Routes
- Paramètres
- Sécurité

## Génération

Génération de documentation pdf.
Génération de code starter.

# Exercices récapitulatifs

## Documentation -> Code

Sur base de la documentation fournie ci-dessous, vous devez développer une API en spring répondant aux besoins demandés.

```yaml
openapi: 3.0.0
info:
  title: API de Gestion de Bibliothèque
  description: API simplifiée pour gérer les livres et les utilisateurs dans une bibliothèque.
  version: 1.0.0

servers:
  - url: http://localhost:8080
    description: Serveur de développement local

paths:
  /books:
    get:
      summary: Récupérer la liste des livres
      parameters:
        - in: query
          name: titleContains
          required: false
          schema:
            type: string
          description: Filtrer les livres dont le titre contient cette chaîne
      responses:
        '200':
          description: Liste des livres
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/Book'

    post:
      summary: Ajouter un nouveau livre
      security:
        - jwt: ['user']
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/Book'
      responses:
        '201':
          description: Livre créé
        '400':
          description: Requête invalide
        '401':
          description: Non autorisé

  /books/{id}:
    get:
      summary: Récupérer un livre par ID
      parameters:
        - in: path
          name: id
          required: true
          schema:
            type: integer
          description: ID du livre
      responses:
        '200':
          description: Livre trouvé
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Book'
        '404':
          description: Livre non trouvé

    delete:
      summary: Supprimer un livre
      security:
        - jwt: ['admin']
      parameters:
        - in: path
          name: id
          required: true
          schema:
            type: integer
          description: ID du livre
      responses:
        '204':
          description: Livre supprimé
        '401':
          description: Non autorisé
        '403':
          description: Accès interdit
        '404':
          description: Livre non trouvé

  /users:
    get:
      summary: Récupérer la liste des utilisateurs
      security:
        - jwt: ['admin']
      responses:
        '200':
          description: Liste des utilisateurs
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/User'
        '401':
          description: Non autorisé
        '403':
          description: Accès interdit

    post:
      summary: Ajouter un nouvel utilisateur
      security:
        - jwt: ['admin']
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/User'
      responses:
        '201':
          description: Utilisateur créé
        '400':
          description: Requête invalide
        '401':
          description: Non autorisé

components:
  schemas:
    Book:
      type: object
      properties:
        id:
          type: integer
        title:
          type: string
        author:
          type: string
        publishedYear:
          type: integer

    User:
      type: object
      properties:
        id:
          type: integer
        username:
          type: string
        email:
          type: string
        role:
          type: string
          enum: ['admin', 'user']

  securitySchemes:
    jwt:
      type: http
      scheme: bearer
      bearerFormat: JWT
      description: Authentification par token JWT
```

Note: Cette documentation a été générée avec une IA. Que pensez vous de la qualité de cette documentation, notamment dans les descriptions fournies ? 

Une documentation ne peut être utile que si elle est parfaitement correcte et répond à un maximum des questions que peut se poser un utilisateur.

## Code -> Documentation

Créez la documentation de l'API disponible à l'adresse [github.com/e-vinci/cae-exercices-exemple](https://github.com/e-vinci/cae-exercices-exemple). N'hésitez pas à récupérer cette API sur votre machine et à la tester pour vous assurer de bien comprendre son fonctionnement.

