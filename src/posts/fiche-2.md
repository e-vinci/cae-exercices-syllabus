---
title: Fiche 2
description: Spring et basés de données.
permalink: posts/{{ title | slug }}/index.html
date: '2026-02-12'
tags: [fiche, spring, orm]
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
      - "5433:5432"
```

Pour rappel, Docker permet de démarrer des "container", sorte de machines virtuelles avec un simple `docker-compose up` dans le répertoire. Vous devez pour cela toutefois avoir le docker dameon actif sur votre machine. En majorité en Windows ceci se fait via [Docker Desktop](https://docs.docker.com/desktop/setup/install/windows-install/).

> Il se peut que l'installation de docker desktop demande "d'upgrader wsl". Ceci peut être fait en ouvrant n'importe quel terminal (cmd ou power shell) avec les droits d'admin puis taper:

```bash
wsl --update
```

> Si vous tentez de faire tourner à la fois le docker fourni et un postgresql sur votre machine, vous risquez des comportement non prédictifs (vu que les deux services vont tourner sur le même port)


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
      - "5433:5432"
```

Ce service va tourner sur le port 5432 - pourquoi alors deux ports ? (`5433:5432`) - parce que la machine dans laquelle va tourner PG n'est pas la vôtre - c'est un container distinct qui a donc ses propres ports. Ceux-ci sont "mappés" aux port de votre machine physique, et peuvent l'être sur des valeurs différentes. Nous rendons PG accessible sur un autre port que celui par défaut pour éviter un problème pour ceux qui ont également postgresql installé sur leur machine.

Dans notre cas on reste au plus simple:

- Faire tourner PG sur le port 5432 sur le container
- Mapper ce port au port 5433 de la machine hôte

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

spring.datasource.url=jdbc:postgresql://localhost:5433/cae_db
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

Générez les getter/setter avec l'IDE. N'utilisez pas de `record` par contre, cela ne va pas fonctionner pour la suite.

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

{% raw %}
```sh
### Read all drinks with File variable
@baseUrl = http://localhost:8080
GET {{baseUrl}}/drinks/
```
{% endraw %}

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

On a rajouté un champs id - dès lors que les "Drink" vont être des lignes dans une base de données, il leur faut une clé primaire. La bonne pratique est de créer un champs dédié pour cela (plutôt que d'utiliser par exemple le nom qui pourrait ne pas être unique). N'hésitez pas à y ajoute un getter si vous voulez voir la valeur retournée par les différents services.

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

L'id fait ici partie intégrante du path (le "chemin" de l'url, soit la partie avant le "?").

[Stucture d'une url](https://cdn.optinmonster.com/wp-content/uploads/2021/10/url-structure.jpg)

Un controller "crud" complet devrait pouvoir gérer ces différentes requests:

- GET /drinks/ => retourne toutes les boissons
- GET /drinks/3 => retourne la boisson avec l'id 3
- POST /drinks/ => créer un nouveau drink avec les infos postées
- PUT /drinks/3 => met à jour la boisson ave l'id 3
- DELETE /drinks3 => supprime la boisson avec l'id 3


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

## Test de la solution

Pour changer on ne va pas vous demander de fournir de fichier http - il est déjà prêt. Vous pouvez vous en servir pour valider votre solution (tous les tests devraient passer). Certaines **assertions** (les client.assert) dépendent de vos données de test - elles devraient fonctionner "telles quelles" si vous avez repris crée les même "drinks" que dans notre exemple plus haut, sinon n'hésitez pas à adapter le fichier http en conséquence.


{% raw %}
```typescript
@baseUrl = http://localhost:8080

### Get the standard message
GET {{baseUrl}}/drinks/

> {%
    client.test("Request executed successfully", function() {
        client.assert(response.status === 200, "Response status is not 200");
    });

    client.test("Response content-type is json", function() {
        var type = response.contentType.mimeType;
        client.assert(type === "application/json", "Expected 'application/json' but received '" + type + "'");
    });

    client.test("Should return all drinkss", function() {
        var body = response.body
        console.log(body);
        client.assert(body.length == 4, "Should return 4 drinks");
    });
%}

### Get the custom message
GET {{baseUrl}}/drinks/1

> {%

    client.test("Should return a drink with id 1", function() {
        var body = response.body
        console.log(body);
        client.assert(body.id == "1", "Should be id 1");
    });
%}

### Create a new drink
POST {{baseUrl}}/drinks/
Content-Type: application/json

{
    "name": "Coke",
    "price": "1.50",
    "description": "Yum Coke",
    "alcoholic": false
}

### Update a drink
PUT {{baseUrl}}/drinks/5
Content-Type: application/json

{
    "id": 5,
    "name": "Pepsi",
    "price": "1.50",
    "description": "Yum Yum Pepsi",
    "alcoholic": false
}

> {%

    client.test("Should update the drink name", function() {
        var body = response.body
        console.log(body);
        client.assert(body.id == "5", "Should be id 1");
        client.assert(body.name == "Pepsi", "Should be id Pepsi");
    });
%}

### Delete a drink
DELETE {{baseUrl}}/drinks/5
```
{% endraw %}

De manière générale, rien ne vous empêcherait d'écrire ce type de tests en premier (avant même d'écrire le controller) - le lancer va logiquement montrer tout des tests qui échouent. Une fois une première méthode écrite (par exemple pour /drinks) dans le controller, l'un des tests devraient passer à vert, et ainsi de suite.

Cette logique d'écrire le test en premier porte un nom - [TDD - "Test Driven Development"](https://www.agilealliance.org/glossary/tdd/).

## Plus loin - des relations

JPA nous permet de mapper facilement une table vers une classe et des lignes vers des objets (dans les deux sens) - maintenant le sens même du modèle **relationnel** ce sont... des relations.

Nous allons voir comment JPA peut gérer des relations entre différents objets (et donc entre des ligne sde tables différentes).

Pour ceci, il nous faut un seconde classe - nos boissons ne sont pas disponible partout, elles sont vendues par des FoodTrucks, chacun actif dans un quartier spécifique.

### FoodTrucks

Avant de parler des relations entre Drinks & FoodTrucks, il nous faut créer:

- Un modèle FoodTruck avec un nom et une adresse (n'oubliez pas les éléments JPA - @Entity at un id)
- Un repository FoodTrucksRepository (simplement créer l'interface)
- Un service avec au moins une méthode pour renvoyer tous les Trucks
- Un controller pour les afficher (à nouveau, un seul endpoint /trucks/ est suffisant pour le moment)

Pour se faciliter les choses, on va créer deux FoodTrucks dans notre Application:

```java
@Bean
    public CommandLineRunner demo(DrinksRepository repository, FoodTrucksRepository foodTrucksRepository) {
        return (args) -> {
            FoodTruck truck1 = foodTrucksRepository.save(new FoodTruck("Chez Momo", "Quartier Saint Boniface"));
            FoodTruck truck2 = foodTrucksRepository.save(new FoodTruck("Ardennes", "Arlon et environs"));
            
            // Création des drinks commé précédemment
```

Tester avec DataGrip que les éléments sont bien insérés dans la base de données
Tester avec le navigateur ou un fichier HTTP que /trucks renvoie bien deux trucks

### Vous avez dit "relations"

Dans notre modèle, nous allons imaginer qu'une boisson n'est disponible que dans un seul camion - chaque camion peut évidemment présenter plusieurs boissons !

On parle d'une relation "ManyToOne" (plusieurs boissons sont disponibles dans un seul camion) - à l'opposition de "OneToOne" et de "ManyToMany". 

Un exemple de relations ManyToMany est par exemple des cours et des étudiants (les étudiants ont plusieurs cours, les cours sont suivi par plusieurs étudiants).

En base de donnée, notre relation pourrait se modéliser avec une clé étrangère (foreign key) de la table drinks vers la table foodtrucks ie:

| id | name         | foodtruck_id |
|----|--------------|--------------|
| 1  | Bloody Mary  | 1            |
| 2  | Mojito       | 1            |
| 3  | Coca         | 1            |
| 4  | Water        | 2            |

Dans ce modèle il est possible de sélectionner toutes les boissons du truck 1 avec:

```sql
SELECT * FROM DRINKS WHERE FOODTRUCK_ID = 1
```

Je vous renvoie à vos cours de base de données pour ces différents concepts.

### Les relations avec JPA

Une relation en JPA n'est jamais qu'un attribut avec une annotation spécifique

```java

@Entity
@Table(name = "drinks")
public class Drink {
    ...

    @ManyToOne
    private FoodTruck foodTruck;
```

La relation est crée du côté "Many" (comme en based de données). On ajoute les getter & les setter comme d'habitude et... c'est tout.

Ceci permet d'adapter nos créations d'objet pour associer un Truck à chaque Drink:

```java
    //Fiche2Application.java
    @Bean
    public CommandLineRunner demo(DrinksRepository repository, FoodTrucksRepository foodTrucksRepository) {
        return (args) -> {
            FoodTruck truck1 = foodTrucksRepository.save(new FoodTruck("Chez Momo", "Quartier Saint Boniface"));
            FoodTruck truck2 = foodTrucksRepository.save(new FoodTruck("Ardennes", "Arlon et environs"));

            repository.save(new Drink("Bloody Mary", "Yum totmatoes", 10.0f, true, truck1));
            repository.save(new Drink("Mojito", "Yum mint", 8.0f, true, truck1));
            repository.save(new Drink("Coca", "Yum sugar", 2.0f, false, truck1));
            repository.save(new Drink("Water", "Yum water", 0.0f, false, truck2));
        };
    }
```

Ceci fait, retournez voir sur la page /drinks/ et vous allez trouver les références aux différents camions.

### Autre direction

On peut donc "suivre" un drink vers son truck (avec drink.foodTruck)... mais pas l'inverse (l'objet Truck n'a pas de référence vers Drink), ce qui serait pratique pour typiquement afficher dans une application toutes les boissons offertes par un camion précis.

```java
@Entity
@Table(name = "foodtrucks")
public class FoodTruck {   
    ...

    @OneToMany(mappedBy = "foodTruck")
    private List<Drink> drinks;
```

On créer un attribute étant une liste de Drink, et on indique que cette relation est gérer de l'autre côté (par l'attribute "foodTruck" dans la classe Drink).

Ne pas oubliez les getters & setters.

Test à nouveau dans le browser et... que voit on ?

### Recursion strikes !

Des lignes de json qui n'en finissent pas, et dans les logs une erreur:

```bash
2025-02-09T10:48:09.424+01:00  WARN 38444 --- [fiche2-pg] [nio-8080-exec-1] .w.s.m.s.DefaultHandlerExceptionResolver : Ignoring exception, response committed already: org.springframework.http.converter.HttpMessageNotWritableException: Could not write JSON: Document nesting depth (1001) exceeds the maximum allowed (1000, from `StreamWriteConstraints.getMaxNestingDepth()`)
2025-02-09T10:48:09.424+01:00  WARN 38444 --- [fiche2-pg] [nio-8080-exec-1] .w.s.m.s.DefaultHandlerExceptionResolver : Resolved [org.springframework.http.converter.HttpMessageNotWritableException: Could not write JSON: Document nesting depth (1001) exceeds the maximum allowed (1000, from `StreamWriteConstraints.getMaxNestingDepth()`)]
```

Quel est le problème ?

- On demande d'afficher le Truck dans le json du Drink
- On demande d'afficher les Drinks dans le json du Truck

Nous avons créer un problème de récursion - le truck1 affiche le drink1 qui lui même affiche le truck1, avec une structure type:

```
- truck1
  - name
  - drinks
    - drink1
      - name
      - truck1
        - name
        - drinks
          - drink1
            ...
```

Le json s'étend à l'infini - et à un moment Spring "tue" le processus pour l'empêcher la page de retour d'être de taille infinie.

Heureusement la solution au problème est simple - indiquer à Spring (plus exactement au package Jackson qui gère la sérialisation des objets vers du json) de quel côté on veut les informations via deux annotations: `@JsonBackReference` & `@JsonManagedReference`

```java
    // FoodTruck.java
    @OneToMany(mappedBy = "foodTruck")
    @JsonManagedReference
    private List<Drink> drinks;
```

```java
    // Drink.java
    @ManyToOne
    @JsonBackReference
    private FoodTruck foodTruck;
```

Le problème devrait être résolu.



