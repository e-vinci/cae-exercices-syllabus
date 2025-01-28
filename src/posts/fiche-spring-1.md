---
title: Fiche 1
description: Introduction a spring.
permalink: posts/{{ title | slug }}/index.html
date: '2025-01-11'
tags: [fiche, spring]
---

# Introduction a Spring

## Objectif et organisation du cours

Ceci est un cours d'introduction à [Spring](spring.io) - un framework web (backend) écrit en Java. Les exercices présents dans les trois fiches de ce cours vont vous servir à apprendre à utiliser le framework pour le projet qui va suivre.

Il ne dure que trois semaines (trois fois quatre heure d'atelier), en mode "travaux dirigés" - chaque semaine a un sujet précis et une fiche d'exercices à suivre.

L'équipe enseigante est présente pour répondre à vos questions ou vous aider en cas de problème.

Le cours en lui même ne donne pas lieu à une évaluation.

## Introduction

### Un framework ?

On distingue générale des "bibliothèques" (librairies ou packages en anglais) des "frameworks". Ceux qui offrent un cadre de développement complet dans lequel notre code application va s'inscrire.

En d'autres mots, on dit souvent qu'on appelle une bibliothèque, mais que dans le cas du framework c'est celui ci qui appelle notre code.

Le but d'un framework est de fournir une structure et une base nécessaire à toutes les applications - il n'y a pas d'intérêt pour une société ou une équipe de réecrire à chaque fois du code permettant par exemple d'envoyer une requête HTTP ou d'accèder à une base de données Postgresql.

Le code dit *applicatif* que nous allons écrire peut alors se conctrer sur les fonctionnalités (features) souhaitées par les utilisateurs.

### Spring

Spring a émergé d'abord comme une alternatives au framework "J2EE" officiel (fourni à l'époque par Sun) percu comme plus facile d'emploi et plus efficace. Le framework s'est depuis imposé comme étant le cadre technique "de facto" des projets backend en Java - il est donc [très présent dans l'industrie](https://survey.stackoverflow.co/2024/technology#1-web-frameworks-and-technologies) y compris en Belgique.

### Backend et frontend

Le cours se concentre exclusivement sur la partie backend - l'idée est que l'on va "supposer" qu'une partie front end complète l'application. En d'autre mots, notre "software" va gérer l'espace de la base de donnée à une API REST - on va donc recevoir et envoyer des requêtes et réponses HTTP en JSON.

Les trois fiches recouvrent la même matière que celle que vous avez vue en Node - mais dans un autre language. La plupart des concepts devraient déjà vous être familier - même si les language et frameworks diffèrent, les logiques générales sont souvent assez similaire (le protocole HTTP reste la base, et Spring, Django, Rails ou autres Laravel sont en fait conceptuellement assez proches)

### Injection de dépendences

## Setup

Pour ce projet nous avons besoins de différents éléments:

- IntelliJ: Disponible sur votre machine, ou à installer (version [Ultimate](https://www.jetbrains.com/idea/download/?section=windows) via compte étudiant Vinci)
- Un JDK Java: Si pas encore présent, IntelliJ va vous proposer d'en installer un - prenez celui [recommandé](https://bell-sw.com/blog/liberica-jdk-21-lts-release-a-lasting-foundation-for-your-java-application/)
- Docker (pour faire tourner des services annexes). Il vous faudra pour cela [Docker Desktop](https://docs.docker.com/desktop/setup/install/windows-install/) sous Windows.

Vérifiez que tout semble bien installé avant de faire la suite.

## Une première application

Nous allons créer une première application Spring Boot. Pour faciliter le setup, Spring Boot dispose d'[Initializr](https://start.spring.io/index.html) - une interface web permettant de créer la structure de base d'un projet en quelques click.

Initializr est inclus dans Intellij nous pouvons donc l'utiliser directement sans quitter l'IDE:

### Créer un nouveau projet

- New Project -> Spring Boot
    - Nom: fiche1
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

Appuyez sur créer.

### Quelques explications

- Language Java: C'est ce que nous faisons ici, mais à noter que la JVM [peut gérer d'autres languages que le Java](https://en.wikipedia.org/wiki/List_of_JVM_languages) - l'exmple le plus fréquent étant Kotlin, mais aussi Scala ou d'autres JRuby. Tous ces languages peuvent être compilés vers un language de plus bas niveau (le [Java Bytecode](https://en.wikipedia.org/wiki/Java_bytecode)) que la JVM peut exécuter
- Maven: Maven est un outil de build - une série de commande et un fichier de configuration (pom.xml) qui permet notemment de:
  - Lister les dépendances (le code externe dans votre application a besoin pour tourner)
  - Gérer des actions ("target") comme compiler, exécuter, lancer les tests sur votre projets ou même le déployer sans avoir à lancer de longuees commandes
- Lombok: Le [Project Lombok](https://projectlombok.org/features/) est un package open source qui fourni un ensemble d'annotations permettant de gagner du temps - par exemple de ne pas avoir à écrire des Getter/Setter ou des fonctions comme [equals ou hashcode](https://projectlombok.org/features/EqualsAndHashCode) à la main 
- Packaging: Une application java va contenir de nombreux fichiers - ceux qui sont réuni dans un fichier .war ou .jar - une sorte de zip avec des méta données spécifiques (vous pouvez d'ailleurs extraire le contenu d'un .jar avec WinZip ou autres - n'hésitez pas à essayer).

### Petit tour du projet

Un ensemble de fichiers et répertoires ont été crées:

- .idea: Répertoire avec des données utilisée par Intellij (configuration de project) - gérer par l'IDE, normalement rien à modifier
- .mvn: Même chose - mais pour Maven
- mvnw: Runner pour maven - permet de lancer facilement des commandes Maven depuis le terminal
- pom.xml: Le fichier principal de maven
- src: Le répertoire où nous allons travailler. Notez que la structure reflète celle du nom du package - src/main/java/be/vinci/cae/...
- .gitgnore: Des instructions pour git de ne pas commiter des fichier générés
- target: le répertoire où seront les fichiers compilés - à nouveau rien à y faire

### Test !

On va ici développer progressivement une application un peu plus grandes que celles auxquelles vous êtes habitués. L'important est de tester le plus souvent possible pour s'assurer que les choses fonctionnent. Il est plus facile de trouver et de corriger une erreur sur cinq lignes de code que sur cinquante ou cinq cent. 

Dans le cas présent on a encore rien codé, mais on peut déjà lancer l'application via le bouton "Play" en haut de l'écran:

![](/images/play-spring-app.png)

Après une série de lignes vous devriez voir un log du type:

```bash
2025-01-17T10:29:25.108+01:00  INFO 3172 --- [fiche1-controllers] [  restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port 8080 (http)
2025-01-17T10:29:25.120+01:00  INFO 3172 --- [fiche1-controllers] [  restartedMain] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2025-01-17T10:29:25.120+01:00  INFO 3172 --- [fiche1-controllers] [  restartedMain] o.apache.catalina.core.StandardEngine    : Starting Servlet engine: [Apache Tomcat/10.1.34]
2025-01-17T10:29:25.154+01:00  INFO 3172 --- [fiche1-controllers] [  restartedMain] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2025-01-17T10:29:25.155+01:00  INFO 3172 --- [fiche1-controllers] [  restartedMain] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 713 ms
2025-01-17T10:29:25.409+01:00  INFO 3172 --- [fiche1-controllers] [  restartedMain] o.s.b.d.a.OptionalLiveReloadServer       : LiveReload server is running on port 35729
2025-01-17T10:29:25.436+01:00  INFO 3172 --- [fiche1-controllers] [  restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port 8080 (http) with context path '/'
2025-01-17T10:29:25.443+01:00  INFO 3172 --- [fiche1-controllers] [  restartedMain] b.v.c.f.Fiche1ControllersApplication     : Started Fiche1ControllersApplication in 1.355 seconds (process running for 1.651)
2025-01-17T10:29:35.839+01:00  INFO 3172 --- [fiche1-controllers] [nio-8080-exec-2] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
2025-01-17T10:29:35.839+01:00  INFO 3172 --- [fiche1-controllers] [nio-8080-exec-2] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
2025-01-17T10:29:35.841+01:00  INFO 3172 --- [fiche1-controllers] [nio-8080-exec-2] o.s.web.servlet.DispatcherServlet        : Completed initialization in 1 ms
```

Dans les choses importantes ici:

 > Completed initialization in 1 ms

indique que le processus s'est bien passé

Un peu plus haut:

> TomcatWebServer  : Tomcat started on port 8080 (http) with context path '/'

Nous indique que le serveur lancé (Tomcat) tourne sur le port 8080 avec un contexte à la racine (/).

Ouvrez un navigateur et pointez le sur localhost:8080. Que voyez vous ? Pourquoi

```
Whitelabel Error Page
This application has no explicit mapping for /error, so you are seeing this as a fallback.

Fri Jan 17 12:06:16 CET 2025
There was an unexpected error (type=Not Found, status=404).
No static resource .
...
```

Ceci est un message d'erreur - mais qui confirme que tout est en place - le serveur tourne bien - mais on ne lui a rien donné à afficher à cet endroit (vous voyez le code d'erreur [HTTP standard 404]() - donc que la page n'est pas trouvée).

La lecture du terminal et des erreurs dans le navigateur est une compétence critique à développer pour comprendre ce qui se passe dans vos applications.

Ceci est un bon moment pour commiter/pusher votre projet sur un repo github avant de poursuivre.

### L'IDE et vous

Intellij nous aide énormément ici - et c'est le but de ces outils ! Gardez toute fois à l'esprit de ne pas trop dépendre d'une "magie" potentiellement incomprise. 

On peut arriver exactement au même endroit sans l'aide de l'IDE (ou avec une autre comme VS Code, etc)

- Générer le projet via [Initializr](https://start.spring.io/index.html) - récupérer le zip et l'extraire dans un folder
- Excuter l'application directement avec Maven:

```bash
     .\mvnw spring-boot:run
```

De manière générale, essayez toujours de regarder ce que fait l'outil derrière un point de menu - la réalité c'est le système de ficiher et ce qui est exécuté par la machine.

## Un premier endpoint

Il est maintenant temps de configurer une url et de renvoyer un message. Pour faire simple nous allons faire un premier "endpoint" (une url exposée par l'API) sur le chemin /hello qui va renovyer un simple message (en JSON):


 /hello
```json
{
    "message": "Hello"
}
```

Pour ce faire nous allons créer un package "controller" et dedans une classe HelloController.java avec le code suivant:

```java
package be.vinci.cae.fiche1.controllers;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/hello")
public class HelloController {

    @GetMapping("/")
    public String hello() {
        return "Hello World!";
    }
}
```

Faite attention au bon placement et nommage des fichiers - Java suppose un lien direct entre les noms de classe et de package et leur position dans le répertoire. Le classe ci-dessus doit donc être dans /src/main/be/vinci/cae/fiche1/controllers/HelloController.java

Si vous avez nommé votre projet/package différent cela peut être une autre localisation. L'important est d'être consistant.

Rouvrez votre navigateur et pointez le sur localhost:8080/hello - vous devriez voir le message "Hello World!".

Prenons un peu de temps pour détailler le code ligne par ligne

```java
@RestController
@RequestMapping("/hello")
public class HelloController {
```

Spring fonctionne avec des annotation plutôt qu'avec un fichier de configuration (par exemple un XML comme utilisé dans des [versions précédentes](https://docs.spring.io/spring-framework/docs/4.2.x/spring-framework-reference/html/xsd-configuration.html)).

Celles ci indiquent:

- Que cette classe ("bean" en termes Spring) est un Controller REST (RestController) - donc que le contenu va être renvoyé "directement", plutôt que par exemple inséré dans un template HTML
- Qu'il va être appelé quand l'url /hello est visitée (RequestMapping)

Au niveau de la méthode:

```java
    @GetMapping("/")
    public String hello() {
```

La méthode dispose de son propre mapping - ici "/" - ce code sera donc exécuté à /hello/ - d'autres méthode de la même classe pourraient être crées avec des mapping different. On note également que c'est un GetMapping - la méthod est donc appellée si on envoie un GET /hello/ (pas un POST ou PUT).

Ne reste qu'a renvoyer le résultat que l'on peut voir à l'écran. Modifiez le string, rechargez la page - le résultat est mis à jour - Spring recharge ici tout le code à chaque visite - c'est donc (un peu plus) lent, mais pratique lorsque l'on développe (cela évite de devoir couper/relancer le serveur à chaque changement).

### Avec du vrai JSON dedans

Notre page renvoie pour l'instant une simple chaine de caractère - hors une API REST implique (souvent) du JSON.

```java
public Map<String,String> hello() {
        Map<String,String> data = new HashMap<>();
        data.put("message", "Hello World in json!");
        return data;
}
```

Petit changement simple - on ne renvoie plus un String mais une Map avec notre message et le tour est joué. C'est Spring Boot (via l'annotation RestController) qui va prendre notre valeur de retour (ici une Map) et la convertir en JSON sans que l'on doive effectuer cela manuellement.

### Requête et paramètres

Nous allong améliorer un peu notre API pour qu'elle puisse souhaiter correctement bonjour à la personne qui fait l'appel. Le but est qu'en faisant /hello/?name=Martin on récupère:

```json
{ "message": "Hello Martin" }
```

Modifions à nouveau notre méthode hello:

```java
public Map<String,String> hello(@RequestParam String name) {
        Map<String,String> data = new HashMap<>();
        data.put("message", "Hello " + name);
        return data;
    }
```
On ajouter un paramètre à la methode - en le préfixant avec l'annotation @RequestParam qui indique à Spring que l'on s'attend à ce que ce paramètre soit fourni via l'url - Spring va alors le récupérer et le passer à la méthode.

Testez votre url avec un nom comme plus haut, puis sans (simplement /hello/) - que se passe-t-il ? Pourquoi ?

Le paramètre est supposé toujours présent - s'il est optionel il faut le spécifier *et gérer le cas où il n'est pas fourni*:

```java
    @GetMapping("/")
    public Map<String,String> hello(@RequestParam(required = false) String name) {
        if (name == null || name.isEmpty()) {
            name = "World";
        }
        Map<String,String> data = new HashMap<>();
        data.put("message", "Hello " + name);
        return data;
    }
```

On peut maintenant appeller l'url avec ou sans paramètre.

Pour information, il est également possible d'écrire ceci avec un [Optional](https://www.baeldung.com/spring-request-param#1-using-java-8-optional)

## Side: Au delà du navigateur

Il est possible de tester notre endpoint de différente manières - la plus simple étant via la ligne de commande avec curl:

```bash
> curl localshot:8080/hello/?name=Leila
{"message":"Hello Leila"}
```

On peut également créer un ficher de test dans Intellij, dans le folder src/test/ nommé hello.http:


{% raw  %}
```http
@baseUrl = http://localhost:8080

### Get the standard message
GET {{baseUrl}}/hello/

### Get the custom message
GET {{baseUrl}}/hello/?name=Sarah
```
{% endraw %}

Une fois le fichier sauvegardé il est possible de lancer ces test avec un bouton de droite > Run All

Intellij va alors exécuter chaque requête présente et sauvegarder les résultats dans des fichiers json. L'avantage sur les méthodes manuelles est que quand l'application devient plus complexe, ces éléments ne doivent pas être réecrit à chaque fois.

Mieux encore il est possible de valider les résultats:

{% raw  %}
```http
@baseUrl = http://localhost:8080

### Get the standard message
GET {{baseUrl}}/hello/

> {%
    client.test("Request executed successfully", function() {
        client.assert(response.status === 200, "Response status is not 200");
    });

    client.test("Response content-type is json", function() {
        var type = response.contentType.mimeType;
        client.assert(type === "application/json", "Expected 'application/json' but received '" + type + "'");
    });

    client.test("No parameters returns Hello World", function() {
        var body = response.body
        console.log(body)
        client.assert(response.body.message == "Hello World", "Cannot find 'World' in body");
    });
%}

### Get the custom message
GET {{baseUrl}}/hello/?name=Sarah

> {%
    client.test("Name parameters returns proper name", function() {
        var body = response.body
        console.log(body)
        client.assert(response.body.message == "Hello Sarah", "Cannot find 'Sarah' in body");
    });
%}
```
{% endraw %}

Ie dans l'ordre:
- Vérifier que la réponse HTTP est bien 200 (donc qu'il n'y a pas d'erreur)
- Vérifier que l'on renvoie bien du json
- Vérifier que ce que l'on recoit est ce qui est attendu (à savoir "Hello World" ou "Hello /name/" si le paramètre est fourni.)

Le concept ici est celui de Tests Unitaires que vous avez sans doute déjà vu (via JUnit ou autre): on fait un appel avec des paramètres fixé et on vérifie que le résultat est confirme à la spécification.

## Un second endpoint

Notre service va en plus du endpoint /hello fournir une liste de citations sur le endpoint /quotes.
Une citation a un titre et un auteur:

> No one returns from a long journey the same person they were before.",
> - Zen Proverb

Notre service va renvoyer 10 quotes quand appelé.

### Controllers

Pour aller au plus simple, nous pouvons déjà créer un Controller et le minimum nécessaire pour tester:

```java
@RestController
@RequestMapping("/quotes")
public class QuotesController {
    @GetMapping("/")
    public List all() {
        return new ArrayList<>();
    }
}
```

Ceci devrait déjà être testable (navigateur, curl ou encore mieux ajouter un quotes.http pour les tests) - quel devrait être le résultat ?

### Modèles

Pour gérer ceci nous allons créer un objet qui représente une citation dans un nouveau package models:

```java
public class Quote {
    String quote;
    String author;

    public Quote() {}

    public Quote(String quote, String author) {
        this.quote = quote;
        this.author = author;
    }

    public String getQuote() {
        return quote;
    }

    public String getAuthor() {
        return author;
    }

    public void setQuote(String quote) {
        this.quote = quote;
    }

    public void setAuthor(String author) {
        this.author = author;
    }
}
```

Le code en question est simple mais verbeux - nous pouvons le rendre beaucoup plus court grâce aux annotation de Lombok - le code suivant est équivalent à celui plus haut:


```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Quote {
    String quote;
    String author;
}
```

Les trois annotations ont chacune un rôle:
- @Data génère des getters, setters hashCode & equals
- @AllArgsConstructor génère un constructeur avec tous les attributes de classe
- @NoArgsConstructor génère un constructeur vide (souvent nécessaire pour la sérialization)

Le code est plus court et surtout plus facile à maintenir - rien n'empêche de créer des setter/getter custom plus tard si le besoin s'en fait sentir.

### Services et injection de dépendances

Nous pourrions placer le code qui crée et renvoie les quotes directement dans le controller - mais les bonnes pratique sont de séparer les rôles:

- Le controller gère les request/response (récupère des paramètre http, renvoie du JSON, etc) donc les aspect "web"
- Le service gère la logique métier

(ou à l'inverse, un controller ne s'occupe pas de logique métier et aucun concept "http" ne devraient être présents dans un service).

On va donc créer un QuoteService pour créer et renvoyer nos Quotes dans un package services:


```java
@Service
public class QuotesService {
    public List<Quote> getAllQuotes() {
        Quote[] quotes = {
                new Quote("Ce qui est écrit n'est pas ce qui est - c'est juste ce qui est écrit", "Anonyme"),
                new Quote("La vie est un mystère qu'il faut vivre, et non un problème à résoudre", "Gandhi"),
                // D'autres quotes
        };

        return List.of(quotes);
    }
}
```

Cet élément est une simple classe Java avec une ou plusieurs méthodes, avec juste l'annotation Service. Nous allons voir son utilité dans un instant.

Reprenons notre QuotesController pour utiliser le service:

```java
@RestController
@RequestMapping("/quotes")
public class QuotesController {
    private final QuotesService quotesService;

    public QuotesController(QuotesService  quotesService) {
        this.quotesService = quotesService;
    }
    @GetMapping("/")
    public Iterable<Quote> all() {
        return quotesService.getAllQuotes();
    }
}
```

On crée un attribute "quotesService" dans le controller, initialisé dans le constructeur et... c'est tout.

Grâce aux différentes annotations (@Service, @Controller), spring va automatiquement créer les instances nécessaires de QuoteController et QuoteService, et s'assurer que l'instance de QuoteService est bien passée au QuotesController à la création.
Retour à /quotes dans le navigateur pour vérifier que l'on voit bien les différentes Quote en JSON.

### Récapitulatif Spring

Durant cette première séance, on a déjà vu pas mal d'avantage du framework:

- Injection de dépendance: On peut créer un ensemble de controllers, services ou autres (on va voir un exemple supplémentaire avec des repository la semaine prochaine) et laisser Spring s'occuper des dépendance.
- Serialisation: On travaille avec des objets java standard (Quote) et les framework les convertis en JSON quand nécessaire.
- On peut "mapper" des méthodes avec des urls et récupérer facilement les paramètres pour s'en servir dans le code.

Au passage on a vu avec Lombok comment écrires des modèles en quelques lignes.

## Exercice complémentaire

Pour récapituler tout ceci, vous allez créer un nouveau endpoint qui renvoie une liste de restaurant.

Un restaurant a:
- Un nom
- Un type de cuisine

L'url /restaurants doit renvoyer 10 restaurants.
Il est possible de passer un paramètre "cuisine" - le endpoint ne doit alors renvoyer que les restaurants de ce type

/restaurants?cuisine=Indien

Il vous faut donc un modèle, un service et un controller. La logique est exactement la même que pour les exercices précédents.

Pour vous aider voici quelques données d'exemple:

```java
String[][] restaurants = {
    {"Comme Chez Soi", "Française"},
    {"Le Chalet de la Forêt", "Belge"},
    {"La Villa Lorraine", "Française"},
    {"Belga Queen", "Belge"},
    {"Bia Mara", "Fish"},
    {"Aux Armes de Bruxelles", "Belge"},
    {"Noordzee Mer du Nord", "Poisson"},
    {"Fin de Siècle", "Européenne"},
    {"Bon Bon", "Française"},
    {"La Quincaillerie", "Belge"},
    {"Café Georgette", "Belge"},
    {"Amadeus", "Ribs"},
    {"Le Pain Quotidien", "Bio"},
    {"The Sister Brussels Café", "Végétarienne"},
    {"Certo", "Italienne"},
    {"Brugmann", "Française"},
    {"Chez Léon", "Belge"},
    {"Yi Chan", "Asiatique"},
    {"Kamo", "Japonaise"},
    {"Humus x Hortense", "Végétarienne"}
};
```

Comme dans les exercices dirigés, soyez sur de progresser par étape et de tester régulièrement. Une démarche possible parmis d'autres:

- Créer le controlleur (avec ses annotations) et renvoyer une simple chaine de caractère. Tester (avec le navigateur)
- Retrouer un array (avec des valeurs fixes) à la place de la chaine. Tester (avec une request http)
- Créer le service avec le même code que le controlleur
- Mettre à jour le controller pour utiliser le service (en utilisant Spring pour injecter la dépendance). Tester que tout fonctionne pareil qu'avant
- Créer le modèle et adapter la méthode du service pour créer des Restaurant plutôt que de simple chaines de caract!res. Tester que vous avez bien les données complètes

De manière générale, éviter d'écrire plus de quelques lignes de code entre chaque test.

