---
title: Fiche 3
description: Spring et sécurité.
permalink: posts/{{ title | slug }}/index.html
date: '2025-01-30'
tags: [fiche, spring]
---

# Spring et JPA

La semaine passée nous avons appris les concepts suivants:

- Persistance des données
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

L'objectif de ce tutoriel va être de protéger les actions qui doivent l'être. Seuls les utilisateurs connectés pourront commander une pizza, et seuls les administrateurs pourront créer, supprimer et éditer des pizzas.

## Project setup

Nous allons cloner un repository contenant le code déjà existant pour le modifier dans cette troisième fiche.

Cloner le repo

Nous allons ajouter des dépendances nécessaires:
- Dans le pom.xml, "edit starters" et ajouter spring security
- Editer le pom.xml pour ajouter java-jwt et spring-dotenv

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

