---
title: Fiche 2
description: Spring et basés de données.
permalink: posts/{{ title | slug }}/index.html
date: '2025-01-18'
tags: [fiche, spring]
---

# Spring et JPA

## Introduction

La semaine passée nous avons crée notre première application Spring, et appris différents concepts:

- Injection de dépendance
- Architecture en couche (Controller, Service)
- Mapping d'url à des méthodes
- Tests d'un API REST via le navigateur et les fichier ".http"

Cette semaine nous allons ajouter une brique supplémentaire - une couche "data" basée sur une base de données relationnelle - pour rappel la semaine passée nos données étaient de simple tableaux en Java (voir un fichier .json).

# Théorie

### But de la séance

Nous allons créer une API pour le point de vente d'un café. Celui-ci propose un ensemble de boissons. Il est donc nécessaire de pouvoir:

- Lister les boissons disponibles (une boisson a un nom, une description, un prix et le fait qu'elle soit alcolisée ou non)
- Recupérer les informations d'une boisson à partir de son identifiant
- Créer une nouvelle boisson
- Supprimer une boisson
- Editer une boisson
- Chercher parmis les boissons basé sur tout ou une partie du nom

Tous ces éléments doivent être crées sous forme de end points.

Ceci constitue une architecture "classique" de backend:

- Couche donnée
- Couche service (logique business)
- Couche web (request/response, etc)

A nouveau cette distinction peut paraître forcée dans l'exercice (on pourrait assez facilement mettre tout le code dans le controller) mais a du sens dès lors qu'on arrive sur des applications de taille réelle (plusieurs dizaines voir centaines de tables, du code en dizaine de millier de lignes ou plus).

### JPA & JDBC

Pour un peu de terminologie:
- JDBC (Java Database Connectivity) permet de connecter une application java à une source de donnée
- JPA - "Java Persistence API" est une *specification* sur la manière dont une application peut se connecter à une base de données. Ce standard permet que des développeurs comme nous puissons facilement utiliser différentes bases de données sans devoir tout réapprendre depuis le début. JPA fonctionne grâce à la couche JDBC
- Hibernate est une implémentation (la plus utilisée) de JPA

Petit schéma pour clarifier:

![](https://velog.velcdn.com/images/blaze241/post/2f6db67e-0918-42b3-8250-111f8224ec87/image.png)

### Repositories

A notre structure Controller/Services/Modèles vient s'ajouter une couche supplémentaire - les Repositories ("dépôts"). Ces classes vont servir de lien entre la base de données (Postgres pour nous) et le reste de l'application.

De la même manière que les Controller sont les seuls endroit où l'on devrait parler de concepts HTTP (Request, Response, Forms, etc), les Repository sont les seules classes qui devraient gérer des aspects base de données - connections, queries, results, etc.

Cette séparation a de nombreux avantages (particulièrement une fois que l'application grandit) - notamment au niveau des tests - la logique business étant dans les Service et indépendante tant des concepts web que de la base de données.

### Entities ("Entités")

Un "Entity" dans JPA est une classe Java (typiquement un modèle) dont les objets ont vocation à être sauvegardés (et lus) dans la base de données.

Donc:

- A une classe (Drink) va correspondre une table dans la base de données
- A un objet (new Drink()...) va correspondre une ligne dans la table

JPA va nous permettre de faire ces opérations - sans écrire une ligne de SQL (ni DDL, ni DML), grâce à quelques annotations.

### Résumé: un ORM

JPA est ce qu'on appelle un ORM: Object Relational Mapper (ORM) - un sytème qui converti des objets en données relationnelles - et l'inverse.

Bien que loin d'être évident à coder, ce n'est en rien de la magie - JPA "traduit" simplement des éléments OO en des éléments relationnels:

- Classes vers Tables
- Objets vers Lignes
- Attribute vers Colonnes

De même, la structure de la classe permet à JPA d'instancier les objets au retour d'un appel à findAll (par exemple).

Comme toute abstraction, bien que très utile, JPA a ses limites et il est important de garder en tête ce qu'il y a "en dessous" - des requêtes SQL essentiellement.

Les modèles objets et relationnels n'ont pas exactement les mêmes capacités - il n'y a en base de données ni héritage ni polymorphisme, etc. 

## Setup

### Postgres

Pour faire fonctionner cet exercice, il nous faut une base de données - Postgresql plus précisément. Si vous n'avez pas de Postgres sur votre machine, vous pouvez simplement faire tourner ce docker-compose.yml (a recopier dans votre projet):

```yml
services:
  db:
    image: postgres:latest
    container_name: postgres_container
    environment:
      POSTGRES_DB: cae_db
      POSTGRES_USER: cae_user
      POSTGRES_PASSWORD: cae
    ports:
      - "5432:5432"
```

Pour rappel, Docker permet de démarrer des "container", sorte de machines virtuelles avec un simple `docker-compose up` dans le répertoire. Vous devez pour cela toutefois avoir le docker dameon actif sur votre machine. En majorité en Windows ceci se fait via [Docker Desktop](https://docs.docker.com/desktop/setup/install/windows-install/).

Docker dépasse le cadre de ce cours, mais de manière simple la configuration indique que l'on souhaite créer un service nommé "db" qui:

```yml
    image: postgres:latest
    container_name: postgres_container
```

se base sur une image existante appellée `postgres` dont on souhaite la dernière version (`latest`). Cette image est récupérée sur Docker Hub (vous pouvez la voir [ici](https://hub.docker.com/_/postgres/)) et téléchargée sur votre machine si vous ne la posséder pas encore.

On va la nommer `posgres_container` (docker compose peut créer et démarrer plusieurs services/container donc les nommer est utile).


```yml
environment:
      POSTGRES_DB: cae_db
      POSTGRES_USER: cae_user
      POSTGRES_PASSWORD: cae
```

Créer un user "cae_user" avec son password "cae" sur une base de données "cae_db" - juste ce qu'il nous faut pour notre projet

```yaml
    ports:
      - "5432:5432"
```

Ce service va tourner sur le port 5432 - pourquoi alors deux ports ? (`5432:5432`) - parce que la machine dans laquelle va tourner PG n'est pas la vôtre - c'est un container distinct qui a donc ses propres ports. Ceux-ci sont "mappés" aux port de votre machine physique, et peuvent l'être sur des valeurs différentes.

Dans notre cas on reste au plus simple:

- Faire tourner PG sur le port 5432 sur le container
- Mapper ce port au port 5432 de la machine hôte

Vous pouvez également lancer le fichier via Intellij directement.

# Exercice 

### Projet

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
- Postgres Driver

Appuyez sur créer.

Nous avons donc deux dépendances supplémentaires:

- Spring Data JPA est un package de Spring avec tout un ensemble de classes et d'annotations pour facilier l'interaction entre le code Java et une base de donnée relationnelle
- Postgres pour avoir les drivers nécessaire à se connecter à notre base de données (chaque base données vient avec ses propres drivers et petites parcularités).

### Petit tour du projet et configuration

La structure devrait vous être familière.

Ouvrez le fichier application.properties et ajoutez les éléments suivants:

```bash
spring.datasource.driver-class-name=org.postgresql.Driver
spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect

spring.datasource.url=jdbc:postgresql://localhost:5432/cae_db
spring.datasource.username=cae_user
spring.datasource.password=cae

spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.generate-ddl=true
```

Attention, si vous avez utilisez des valeurs différentes pour le nom ou user de la base de données, il faut adapter en conséquence !

JPA permettant de se connecter à quasiment n'importe quelle base de données nous devons spécifier un ensemble d'éléments.

- `driver-class-name` indique quelle classe utiliser pour intéragir avec la base de données (chaque base de donnée a son propre driver)
- Le dialecte est en hibernate (l'implémentation de JPA)

- `datasource-url` donne le "chemin" vers la base de donnée. Ici jdbc (la connection) : postgresql (le type de base de données) : mem (en mémoire) : cae_db (le nom de notre DB).
- On trouve après classiquement un user & password

Les deux derniers éléments indiquent à JPA de créer ou droper (supprimer) les tables et schéma en fonction de ce qui est défini dans le code (on va y revenir).

## Controller, Service, Model

On va repartir de la même logique que celle utilisée la semaine passée 

### Modèle

Nous allons créer un modèle Drink, représenté de manière simple ici.


```java
public class Drink {
    public Drink() {}

    public Drink(String name, String description, float price, Boolean alcoholic) {
        this.name = name;
        this.description = description;
        this.price = price;
        this.alcoholic = alcoholic;
    }

    private String name;
    private String description;
    private float price;
    private Boolean alcoholic;
}
```

A vous de voir si vous générez les getter/setter ou si vous passez par Lombok, ca ne fait pas de différence (pas de `record` par contre, cela ne va pas fonctionner pour la suite).

### Service

Un service pour récupérer "tous" les Drink - au moins quelques-uns pour tester.

```java
package be.vinci.cae.fiche2.services;

import be.vinci.cae.fiche2.models.Drink;
import org.springframework.stereotype.Service;

@Service
public class DrinksService {
    public Iterable<Drink> getAllDrinks() {
        List<Drink> allDrinks = List.of({
            new Drink("Bloody Mary", "Yum totmatoes", 10.0f, true),
            new Drink("Mojito", "Yum mint", 8.0f, true),
            new Drink("Water", "Fresh!", 2.0f, false)
        });

        return allDrinks;
    }
}
```

`List.of` permet d'initialiser une liste à partir d'un tableau.

### Controller

Comme précédemment - on va se limiter pour l'instant à l'endpoint "all":

```java
package be.vinci.cae.fiche2.controllers;

import be.vinci.cae.fiche2.models.Drink;
import be.vinci.cae.fiche2.services.DrinksService;

@RestController
@RequestMapping("/drinks")
public class DrinksController {
    private DrinksService drinksService;

    public DrinksController(DrinksService drinksService) {
        this.drinksService = drinksService;
    }

    @GetMapping("/")
    public Iterable<Drink> getDrinks() {
        return drinksService.getAllDrinks();
    }
}
```

Temps de tester tout ceci (via le navigateur) - et de créer un petit fichier http pour facilement vérifier les résultats:

```http
### Read all drinks with File variable
@baseUrl = http://localhost:8080
GET {{baseUrl}}/drinks/
```

Nous sommes plus ou moins de retour au résultat précédent - on va maintenant introduire nos Entités et Repository.

## Entities and Repositories

### Entity 

Nous allons reprendre notre modèle "Drink" et en faire une entité.

```java
@Entity
@Table(name = "drinks")
public class Drink {
    ...

    @Id
    @GeneratedValue
    private Long id;
    ...
```

Au niveau de la classe, l'annotation `@Entity` indique que le modèle doit correspondre à une table.
`@Table` permet de spécifier le nom de la table (si on ne le fait pas, JPA a des "convention" qui vont créer le nom basé sur le nom de la classe).

On a rajouté un champs id - dès lors que les "Drink" vont être des lignes dans une base de données, il leur faut une clé primaire. La bonne pratique est de créer un champs dédié pour cela (plutôt que d'utiliser par exemple le nom qui pourrait ne pas être unique).

L'annotation @Id indique que ce champs n'est pas un champs "comme les autres" - mais bien la clé primaire. `@GeneratedValue` indique que c'est la base de donnée qui va mettre cette valeur lorsqu'on fait un INSERT.

Certaines autres annotations peuvent être utilisée sur le différents champs comme par exemple `@Column` - mais ce n'est pas obligatoire. JPA va supposer que chaque champs de l'objet correspond à une colonne, et va adapter les types en conséquences (String vers Varchar, etc):

```java
@Column(unique=true)
private String name;
```

L'exemple ici indique que le nom doit être unique (générant une contrainte dans la base de donnée).

### Repository

Un `Repository` est un type de Component Spring (comme `@Service` et `@Controller` que nous avons déjà rencontré) - le rôle de repository est justement de gérer les interaction entre les objets et la base de donnée.

Nous allons créer notre `DrinksRepository` (dans un package `repositories`)

```java
@Repository
public interface DrinksRepository extends CrudRepository<Drink, Long> {
}
```

Ce qu'on défini ici est une interface... et en plus il n'y a aucun code (à part une extension) ?

Temps de tester ceci via un nouveau concept au niveau de notre application.

### CommandLineRunner 

On va ouvir l'application que Spring nous a créer (Fiche2JpaApplication chez moi) et rajouter dans une méthode dans la classe:

```java
...
@Bean
public CommandLineRunner demo(DrinksRepository repository) {
	return (args) -> {
        System.out.println("Hello!");
    };
}
```

Le [CommandLineRunner](https://docs.spring.io/spring-boot/api/java/org/springframework/boot/CommandLineRunner.html) est une interface qui indique que le code présent doit être éxecuté au lancement de l'application. C'est un option très pratique pour tout un ensemble de traitement - comme charger des données.

Relancez l'application - vous devriez voir le "Hello" dans les logs.

### Seed data and the ORM

Nous allons adapter le CommandRunner pour insérer des données dans une table - grâce au modèle et au repository.

Un peu de code avant les explications:

```java
public CommandLineRunner demo(DrinksRepository repository) {
    return (args) -> {
        repository.save(new Drink("Bloody Mary", "Yum totmatoes", 10.0f, true));
        repository.save(new Drink("Mojito", "Yum mint", 8.0f, true));
        repository.save(new Drink("Coca", "Yum sugar", 2.0f, false));
        repository.save(new Drink("Water", "Yum water", 0.0f, false));
```

Relancez le serveur et regardez dans les logs s'ils y a des erreurs (cela ne devrait pas).

### Test !

Ouvrez votre base de données dans DataGrip (ou via n'importe quel outil permettant de s'y connecter)

Vous devriez voir la base de données avec la table drinks. Faite rapidement un SELECT * FROM DRINKS - vous devriez voir les différents records.

Pour s'assurer de ce qui se passe, nous allons ajouter un peu de log via deux lignes de plus dans application.properties

```bash
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```

Ceci demande à Spring Boot de logger tous les SQLs.

Relancez l'applications et regardez les logs vous devriez voir quelque chose comme ceci:

```sql
Hibernate: 
    create table drinks (
        alcoholic boolean,
        price float(24) not null,
        id bigint not null,
        description varchar(255),
        name varchar(255) unique,
        primary key (id)
    )
```

soit la création d'une table drinks, puis un peu plus loin:

```sql
Hibernate: 
    select
        next value for drinks_seq
Hibernate: 
    insert 
    into
        drinks
        (alcoholic, description, name, price, id) 
    values
        (?, ?, ?, ?, ?)
```

soit un appel à une séquence (une fonctionnalité des base de données relationnelle permettant de générer un id) puis un INSERT.

Mais comment ces insertions ont elles été effectuées ? Il faut retourner dans le modèle et le controller pour bien comprendre ce que fait JPA "under the hood" ("en dessous du capot").

### Retour sur les modèles et repository

Donc nous avons défini un modèle avec l'annotation @Entity et le nom de la table. Ces informations, et les attributes de la classe Drink permettent à JPA de générer un DDL comme vu plus haut.

Grâve au dialecte, il sera généré dans le DDL de la base de données choisie (SQL n'est pas complètemeent standard, donc c'est important !).

La seconde partie (les insertions) vient du Repository donc cette simple ligne:

```java
@Repository
public interface DrinksRepository extends CrudRepository<Drink, Long> {
}
```

Le [CrudRepository](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/CrudRepository.html) fourni par défaut les méthodes standard sont on a besoin pour un modèle:

- findAll() => SELECT * FROM DRINKS
- findById(id) => SELECT * FROM DRINKS WHERE ID = id
- save() => INSERT ...
- update() => UPDATE FROM...
- delete() => DELETE FROM...

Ces méthodes sont donc disponible pour le modèle sans avoir besoin de les coder - vu qu'à part le nom de la classe et des modèle, le SQL est toujours le même - JPA est donc capable de le générer.

On peut évidemment rajouter des méthodes sur le repository - encore mieux, si ces méthodes sont bien nommées, il ne faut même pas les implémenter:

```java
@Repository
public interface DrinksRepository extends CrudRepository<Drink, Long> {
    Drink findByName(String name);
}
```

`name` étant un attribute connu sur `Drink`, JPA peut générer le contenu de la méthode - et donc un SQL du type:

```sql
SELECT * FROM DRINKS WHERE name = ?
```


## End to End List

On peut maintenant mettre toutes les pièces ensemble - on va injecter le `DrinksRepository` dans le `DrinksService` pour pouvoir s'en servir pour récupérer nos données à la place de celles hard codées:

```java
@Service
public class DrinksService {
    private final DrinksRepository drinksRepository;

    public DrinksService(DrinksRepository drinksRepository) {
        this.drinksRepository = drinksRepository;
    }

    public Iterable<Drink> getAllDrinks() {
        return drinksRepository.findAll();
    }

    ...
}
```

Testez à nouveau (.http ou dans le navigateur), tout devrait bien fonctionner - pas besoin de changer le controller - que les données arrivent de constante ou de la base de donneé ne fait pas de différence.

## CRUD

Nous allons maintenant compléter notre API pour les différentes fonctionss "CRUD". Côté controller, en suivant les standards REST on veut quelque chose comme: 

- GET /drinks: Récupérer toutes les boissons
- GET /drinks/:id: Récupérer une boisson avec cet id là
- POST /drinks: Créer une nouvelle boisson
- PUT /drinks/:id: Mettre à jour la boisson
- DELETE /drinks/:id: Supprimer la boisson

Cela correspond assez directements aux action "CRUD" dans la base de données:

- SELECT *
- SELECT * WHERE ID = :id
- UPDATE WHERE ID = :id
- DELETE WHERE ID = :id

### Controller

Un petit update côté Controller - on a vu comment gérer des paramères dans l'url

> http://localhost:8080/drinks?name=bloody

avec quelque chose comme:

```java
@GetMapping("/")
public String hello(@RequestParam(required = false) String name) { ...
```

On peut voir une logique similaire pour des paramêtres qui font partie de l'url avec @PathVariable

> http://localhost:8080/drinks/1

```java
@GetMapping("/{id}")
public String getDrink(@PathVariable() String id) { ...
```

### Repository 

Le `DrinksRepository` peut déjà faire toutes ces opérations - nous n'avons qu'à les appeller correctement depuis le service.

```java
    public Drink getDrink(long id) {
        return drinksRepository.findById(id).orElse(null);
    }


    public Drink createDrink(Drink drink) {
        return drinksRepository.save(drink);
    }

    public void deleteDrink(long id) {
        drinksRepository.deleteById(id);
    }
```

Notez cette ligne:

```java
    return drinksRepository.findById(id).orElse(null);
```

et particulièrement le `orElse` - que fait cette méthode?

La réponse est dans la définition de `findById` tel que défini dans le `CrudRepository` de Spring documentée [ici](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/CrudRepository.html).

La signature est:

```java
Optional<T> findById(ID id)
```

Le `<T>` indique que cette fonction est générique - elle retourne une valeur du type défini - notre repository étant défini comme un `CrudRepository<Drink>`, la méthode renvoie logiquement un objet de type Drink... ou plus exactement un `Optional<Drink>`.

Pourquoi ? Cette méthode va générer un `SELECT * FROM DRINKS WHERE ID = :id` - ce qui va envoyer un record correspondant à un Drink... ou pas de record du tout (si l'id n'est pas présent dans la table).

La méthode renvoie donc un [Optional](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html) - un objet qui contient soit une valeur (le Drink) - soit null.

Ce "pattern" permet de facilement tester si la valeur existe ou non. orElse() est une méthode d'Optional qui renvoie soit la valeur soit null si absente.

Créer des fichiers http pour tester le tout.