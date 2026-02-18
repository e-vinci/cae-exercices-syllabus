---
title: Fiche 5
description: Qualité backend et Intégration continue
permalink: posts/{{ title | slug }}/index.html
date: '2026-02-20'
tags: [fiche, spring, quality, ci]
---

# Gestion de la qualité du backend et Intégration continue

La fiche de cette semaine couvre 2 aspects importants qui permettent d'améliorer la qualité de vos développements:
- amélioration de la qualité du code "backend" 
- adoption de bonnes pratiques de développement et de l'Intégration Continue

# 1. Comment gérer la qualité du code backend

La qualité de l'API est essentielle pour offrir une expérience optimale aux développeurs et maintenir la cohérence et la maintenabilité du code. 

Voici quelques pratiques clés pour gérer la qualité de l'API :
- **Style guide de Google** : Utiliser le style guide de Google pour les conventions de code et de nommage. Cela permet de maintenir un code propre, lisible et cohérent à travers toute l'équipe de développement.

- **Tests statiques** : Utiliser des outils d'analyse statique pour vérifier la qualité du code, détecter les vulnérabilités de sécurité et appliquer les normes de codage. (Par exemple, avec Java, on peut utiliser des outils comme `Checkstyle`, `PMD` et `CPD`)

- **Architecture modulaire** du code : Adopter une architecture modulaire pour le code de l'API. (Avec Spring Boot, il suffit de découper le code en `repository`, `service`, `controller` et `model`)

- **Tests unitaires** avec JUnit & Mockito : Écrire des tests unitaires pour vérifier le bon fonctionnement de la logique de l'application.

- **Tests d'intégration** avec Spring Boot : A l'aide de requêtes HTTP, on peut facilement tester l'intégration des différentes parties de l'API.



## Comment résumer le style guide de Google ?

Voici un minuscule résumé de quelques points du style guide de Google pour Java :

- **Nommage des variables et des fonctions** :
    - Utiliser le camelCase pour les noms de variables et de méthodes.
    - Utiliser PascalCase pour les noms de classes et d'interfaces.
    - Les constantes doivent être en UPPER_SNAKE_CASE.

- **Formatage du code** :
    - Utiliser des espaces pour l'indentation, avec une taille de tabulation de `2` ou `4` espaces.
    - Respecter une largeur de ligne maximale de `100` caractères.
    - Placer les accolades ouvrantes `{` à la fin de la ligne de déclaration et les accolades fermantes `}` sur une nouvelle ligne.

- **Commentaires** :
    - Utiliser des commentaires Javadoc (`/** ... */`) pour documenter les classes, les méthodes et les champs publics.
    - Utiliser des commentaires en ligne (`// ...`) pour expliquer des sections spécifiques du code.

- **Organisation des imports** :
    - Grouper les imports par type : imports standard de Java, imports de bibliothèques tierces, imports internes.
    - Éviter les imports globaux (`import ....*`).

- **Structure des classes** :
    - Suivre l'ordre suivant pour les membres d'une classe : variables de classe (statique), variables d'instance, constructeurs, méthodes publiques, méthodes protégées, méthodes privées.
    - Utiliser des modificateurs d'accès appropriés (public, protected, private) pour encapsuler les données et les méthodes.

Plutôt que de devoir apprendre les règles, le plus simple est de mettre en place le checkstyle au sein d'IntelliJ afin de checker le code selon le style guide de Google.

## Comment mettre en place le Checkstyle dans IntelliJ ?

Pour ce tutoriel, veuillez créer un nouveau projet nommé `api` sur base d'un copier/coller du dossier `api-starter` ici : [api-starter](https://github.com/e-vinci/cae-theory-demos/tree/main/api-starter).

Pour démarrer votre API, vous devez auparavant avoir une DB Postgres qui tourne localement.
Pour cela, vous devez avoir installé `Docker Desktop` sur votre machine. :

Veuillez ouvrir un terminal au sein du répertoire **`/api`** de votre projet et exécuter la commande suivante :
```bash
docker compose up --build
```

Maintenant que votre DB est exécutée, vous pouvez démarrer votre API à l'aide d'IntelliJ.

Pour mettre en place le Checkstyle dans IntelliJ, suivez les étapes suivantes :
- Allez dans les paramètres d'IntelliJ (`IntelliJ IDEA` > `Settings` > `Plugins` > `Marketplace`).
- Téléchargez le plugin `Checkstyle-IDEA` depuis le *Marketplace* d'IntelliJ.
- Redémarrez IntelliJ pour activer le plugin.
- Allez dans les paramètres d'IntelliJ (`IntelliJ IDEA` > `Settings`).
- Cherchez `Checkstyle` dans la barre de recherche.
- Sélectionnez dans `Configuration File` : `Google Checks`.
- Cliquez sur `OK` pour valider la configuration.

Pour que le formatage automatique d'IntelliJ applique les règles de style de Google, allez dans les paramètres d'IntelliJ : `IntelliJ IDEA` > `Settings` > `Editor` > `Code Style` et sélectionnez `GoogleStyle` dans le menu déroulant `Scheme`.  
Que faire si `GoogleStyle` n'apparaît pas dans le menu déroulant `Scheme` ?
- Veuillez télécharger le fichier https://github.com/google/styleguide/blob/gh-pages/intellij-java-google-style.xml (cliquez sur l'icône `Download raw file` sur github).
- Dans `IntelliJ IDEA` > `Settings` > `Editor` > `Code Style`, cliquez sur la roue crantée, puis `Import Scheme`, puis `IntelliJ IDEA code style XML` et sélectionnez le fichier `intellij-java-google-style.xml` que vous venez de télécharger. `GoogleStyle` devrait maintenant apparaître dans le menu déroulant `Scheme`.

Certains soucis de style seront maintenant signalés dans votre code Java. Vous pouvez les corriger :
- En utilisant le formatage automatique d'IntelliJ (`Ctrl + Alt + L` ou `Code`, `Reformat Code`).
- En utilisant les suggestions d'IntelliJ pour corriger les problèmes de style guide.

Vous avez maintenant accès à un nouvel onglet `Checkstyle` (dans la barre à gauche, un peu avant l'onglet `Terminal`) dans IntelliJ qui vous permet de visualiser les problèmes de style dans votre code Java.

Sélectionnez le fichier `PizzaService`. Cliquez sur l'onglet `Checkstyle` pour voir les problèmes de style dans votre code Java. Vous pouvez checker le style soit pour le fichier courant, soit pour tout le projet :
- Cliquez sur le bouton `Play` pour lancer le checkstyle pour le fichier courant.
- Cliquez sur le bouton `Check Project` pour lancer le checkstyle pour tout le projet.
  Le Checkstyle a trouvé 393 "Warnings" à corriger dans le projet. 

## Quelle bonne dose de commentaires dans du code Java ?

### Généralités

L'idée générale est d'avoir **juste assez de commentaires** pour rendre le code compréhensible, mais pas trop au point de le surcharger. Un bon commentaire doit **ajouter de la valeur** et ne pas répéter ce qui est déjà évident dans le code.
- **Trop de commentaires** → Risque de noyer l'essentiel et d'alourdir la lecture.
- **Pas assez de commentaires** → Le code peut devenir obscur pour d'autres développeurs (ou toi-même dans quelques mois).

### Bonne pratiques

**Priorité au code lisible**
- Des **noms de variables** et **méthodes explicites** évitent de devoir commenter.

**Commentaires uniquement quand ils ajoutent de la valeur**
- **Expliquer le pourquoi, pas le quoi**(expliquer pourquoi une chose est faite, pas ce qu’elle fait.)
- **Clarifier un algorithme complexe**
- **Documenter une API publique avec Javadoc**

**Éviter les commentaires inutiles**
- Pas besoin de commenter du code trivial.
- Si un commentaire explique **quelque chose qui peut être clarifié en renommant une variable ou en refactorisant le code**, il est inutile.

## Comment automatiser l'exécution du Checkstyle ?

Nous allons voir comment mettre en place le plugin `Checkstyle` pour Maven afin de vérifier le respect des conventions de code dans votre projet Java. Nous souhaitons pouvoir utiliser `Maven` car nous voulons nous diriger vers une automatisation du processus de vérification du style de code.

Il faut commencer par ajouter le fichier `google_checks.xml` à la racine de votre projet. Ce fichier contient les règles de style à appliquer pour le projet. Vous pouvez le trouver [ici](https://raw.githubusercontent.com/checkstyle/checkstyle/master/src/main/resources/google_checks.xml).

On veut faire les checks de style que lors des tests : lorsque l'on exécute **`mvn test`**, on veut que les checks de style soient effectués.

Au sein de **`<plugins>`** dans **`pom.xml`**, ajoutez la dépendance suivante :

```xml
<plugin>
   <groupId>org.apache.maven.plugins</groupId>
   <artifactId>maven-checkstyle-plugin</artifactId>
   <version>3.6.0</version>
   <executions>
      <execution>
      <phase>test</phase>
      <goals>
         <goal>check</goal>
         <goal>checkstyle</goal>
      </goals>
      </execution>
   </executions>
   <configuration>
      <configLocation>google_checks.xml</configLocation>
      <consoleOutput>true</consoleOutput>
      <failsOnError>true</failsOnError>
      <violationSeverity>warning</violationSeverity>
      <maxAllowedViolations>10</maxAllowedViolations>

   </configuration>
   <dependencies>
      <dependency>
      <groupId>com.puppycrawl.tools</groupId>
      <artifactId>checkstyle</artifactId>
      <version>10.20.0</version>
      </dependency>
   </dependencies>
</plugin>
```

Veuillez exécuter la commande suivante au sein d'un terminal d'IntelliJ pour vérifier le respect des conventions de code dans votre projet Java :

```sh
mvn test
```

Voici le message que l'on reçoit :
**`[ERROR] Failed to execute goal org.apache.maven.plugins:maven-checkstyle-plugin:3.6.0:check (default) on project cae-exercices-demo: You have 393 Checkstyle violations. The maximum number of allowed violations is 10. `**

On voit que le checkstyle a trouvé 393 violations à corriger dans le projet. De plus, on voit que le checkstyle a été configuré pour ne pas autoriser plus de 10 violations.

Si l'on souhaitait être plus ou moins tolérant pour le nombre de violations, on pourrait modifier la valeur de **`<maxAllowedViolations>`** dans le fichier **`pom.xml`**.

Ce qui est actuellement dérangeant, c'est que Maven ne génère pas de rapport HTML pour les violations de style s'il considère que le build doit échouer (ce qui est le cas étant donné que l'on dépasse le nombre de violations autorisées).

## Comment utiliser un profile avec Maven ?

Nous allons améliorer la configuration du projet Maven afin de générer un rapport HTML des violations de style, même si le build échoue.

Après la section **`<build>`** dans **`pom.xml`**, toujours au sein de **`<project>`**, nous allons créer un **`profile`** pour générer le rapport HTML des violations de style, peu importe si le build échoue ou non.

Un profil Maven est un ensemble de configurations qui permet de personnaliser le comportement du build en fonction de différents environnements ou conditions. Les profils peuvent être utilisés pour activer ou désactiver certaines parties de la configuration du projet, comme les plugins, les dépendances, les propriétés, etc.

Les profils sont définis dans le fichier `pom.xml` sous l'élément `<profiles>`. Chaque profil est identifié par un `<id>` unique et peut contenir des configurations spécifiques pour le build.

Les profils peuvent être activés ou désactivés de différentes manières :

- **Activation par ligne de commande** :
    - Vous pouvez activer un profil en utilisant l'option **`-P`** de la ligne de commande Maven.
    - Exemple : **`mvn clean install -Pmy-profile`**

- **Activation par propriété** :
    - Vous pouvez activer un profil en définissant une propriété spécifique.
    - Exemple :
      ```xml
      <activation>
        <property>
          <name>env</name>
          <value>production</value>
        </property>
      </activation>
      ```
    - Commande : `mvn clean install -Denv=production`

- **Activation par JDK** :
    - Vous pouvez activer un profil en fonction de la version du JDK utilisé.
    - Exemple :
      ```xml
      <activation>
        <jdk>1.8</jdk>
      </activation>
      ```

- **Activation par système d'exploitation** :
    - Vous pouvez activer un profil en fonction du système d'exploitation.
    - Exemple :
      ```xml
      <activation>
        <os>
          <name>Windows 10</name>
        </os>
      </activation>
      ```

Dans notre cas, nous allons utiliser la première option, c'est-à-dire activer le profil en utilisant l'option **`-P`** de la ligne de commande Maven.

Veuillez ajouter ce profil Maven à la fin du fichier **`pom.xml`** (juste avant la balise de fermeture **`</project>`**, en dehors de la balise **`<build>`**) :

```xml
  <profiles>
    <profile>
      <id>no-build-failure</id>
      <build>
        <plugins>

          <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-checkstyle-plugin</artifactId>
            <version>3.6.0</version>
            <configuration>
              <consoleOutput>true</consoleOutput>
              <failsOnError>false</failsOnError>
              <failOnViolation>false</failOnViolation>
            </configuration>
          </plugin>        

        </plugins>
      </build>
    </profile>
  </profiles>
```

Lorsque ce profil est activé, le plugin `maven-checkstyle-plugin` est configuré pour ne pas échouer le build en cas de violations de style. Cela permet de générer un rapport HTML des violations de style, même si le build échoue.

Veuillez exécuter la commande suivante au sein d'un terminal d'IntelliJ pour générer un rapport HTML des violations de style dans votre projet Java :

```sh
mvn clean test -P no-build-failure
```

Nous avons maintenant un rapport HTML des violations de style généré dans le répertoire **`/target/reports/checkstyle.html`**. Vous pouvez ouvrir ce fichier dans un navigateur pour visualiser les violations de style dans votre projet Java.

Pour ce faire, vous pouvez faire un clic droit sur le fichier **`checkstyle.html`** dans l'arborescence du projet d'IntelliJ, puis sélectionner **`Open in Browser`**.

OK, il est temps de supprimer les 393 violations de style dans votre projet Java. Vous pouvez utiliser les suggestions d'IntelliJ pour corriger les problèmes de style guide.

Une façon d'aller plus vite : faite un clic droit sur le dossier `/src/main/java` de votre projet, puis sélectionnez **`Reformat Code`** en sélectionnant toutes les options (`Include subdirectories`, `Optimize imports`, `Rearrange entries`, `Only changes uncommitted to VCS`, `Cleanup code`). Cela va appliquer automatiquement les règles de formatage de code définies dans le fichier **`google_checks.xml`**.

En cliquant sur `Check Project` de l'onglet `Checkstyle` dans IntelliJ, vous devriez voir que le nombre de violations a diminué. Dans mon cas il en reste 89 à corriger.

Allez-y, vous pouvez le faire, corrigez tout cela manuellement ! 🚀
N'hésitez pas à utiliser Github Copilot pour générer la JavaDoc des méthodes, des classes et des interfaces.

Une fois plus aucune erreur, veuillez exécuter les tests et voir si vous avez bien un rapport montrant que tout est en ordre :

```sh
mvn clean test 
```

## Comment mettre en place le PMD ?

Nous allons voir comment mettre en place le plugin `PMD` pour Maven afin de détecter les violations de code dans votre projet Java.

PMD se concentre sur la détection des erreurs de programmation et des mauvaises pratiques de codage, comme les duplications de code, les variables non utilisées, les méthodes trop longues, etc.

Pour commencer, ajoutez le plugin PMD à votre fichier `pom.xml` dans la section `<plugins>` :

```xml
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-pmd-plugin</artifactId>
        <version>3.26.0</version>

        <configuration>

          <failOnViolation>true</failOnViolation>
          <printFailingErrors>true</printFailingErrors>
          <minimumTokens>40</minimumTokens>

          <rulesets>
            <ruleset>rulesets/java/quickstart.xml</ruleset>
          </rulesets>

        </configuration>
        <executions>
          <execution>
            <id>pmd-check</id>
            <phase>test</phase>
            <goals>
              <goal>check</goal>
            </goals>
            <configuration>
              <maxAllowedViolations>0</maxAllowedViolations>
            </configuration>
          </execution>
          <execution>
            <id>pmd-cpd-check</id>
            <phase>test</phase>
            <goals>
              <goal>cpd-check</goal>
            </goals>
            <configuration>
              <maxAllowedViolations>0</maxAllowedViolations>
            </configuration>
          </execution>
        </executions>
      </plugin>
```

**`<minimumTokens>40</minimumTokens>`** : ce paramètre définit le nombre minimum de tokens qui doivent être dupliqués pour que CPD considère cela comme une duplication de code.

Le ruleset `rulesets/java/quickstart.xml` contient des règles de base pour détecter les erreurs de programmation courantes.

Voici une brève description d'autres rulesets PMD que nous aurions pu intégrer :
- **basic.xml**: Contient des règles de base pour détecter les erreurs courantes de programmation.
- **braces.xml** : Vérifie l'utilisation correcte des accolades dans le code.
- **unusedcode.xml**: Détecte le code non utilisé, comme les variables et méthodes inutilisées.
- **design.xml** : Contient des règles pour améliorer la conception du code.
- **codesize.xml** : Vérifie la taille du code, comme la longueur des méthodes et des classes.
- **naming.xml** : Vérifie les conventions de nommage pour les classes, méthodes et variables.
- **optimizations.xml** : Contient des règles pour optimiser le code.
- **strings.xml** : Vérifie l'utilisation correcte des chaînes de caractères.
- **controversial.xml** : Contient des règles controversées qui peuvent ne pas être acceptées par tous les développeurs.
- **migrating.xml** : Contient des règles pour aider à la migration entre différentes versions de Java.
  multithreading.xml : Vérifie les problèmes potentiels liés à la programmation multithread.


Puis, rajoutez cette partie dans le profile `no-build-failure` au sein de la balise `<profiles>` :

```xml
<plugin>
   <groupId>org.apache.maven.plugins</groupId>
   <artifactId>maven-pmd-plugin</artifactId>
   <version>3.26.0</version>

   <executions>
      <execution>
         <id>pmd-check</id>
         <phase>test</phase>
         <goals>
         <goal>check</goal>
         </goals>
         <configuration>
         <failOnViolation>false</failOnViolation>
         </configuration>
      </execution>
      <execution>
         <id>pmd-cpd-check</id>
         <phase>test</phase>
         <goals>
         <goal>cpd-check</goal>
         </goals>
         <configuration>
         <failOnViolation>false</failOnViolation>
         </configuration>
      </execution>
   </executions>
   </plugin>
```

Cela permet de configurer le plugin PMD pour ne pas échouer le build en cas de violations de code. Cela permet de générer un rapport HTML des violations de code, même si le build échoue.

Veuillez exécuter la commande suivante au sein d'un terminal d'IntelliJ pour détecter les violations de code dans votre projet Java :

```sh
mvn test -P no-build-failure
```

Vous devriez voir un rapport HTML des violations de code généré dans le répertoire **`/target/reports/pmd.html`**. Vous pouvez ouvrir ce fichier dans un navigateur pour visualiser les violations de code dans votre projet Java.

PMD a détecté 2 violations de code dans notre projet, dans les classes **`DemoApplication.java`** et **`JwtAuthenticationFilter.java`**. Vous pouvez ouvrir le rapport HTML pour voir les détails des violations et les recommandations pour les corriger.

Pour **`DemoApplication.java`**, PMD signale une violation de la règle `UseUtilityClass	: This utility class has a non-private constructor`. Cela signifie que la classe `DemoApplication` est une classe utilitaire mais qu'elle a un constructeur non privé. Pour corriger cette violation, vous pouvez rendre le constructeur de la classe `DemoApplication` privé. Le souci si on rend le constructeur privé, c'est que Spring ne pourra plus instancier la classe, ou, que PMD s'attendra à ce que la class soit `final`. Et donc, si l'on rend la classe `final`, nous ne pourrons pas compiler notre application.  
Au final, dans ce cas particulier, PMD se trompe. Il est donc préférable de désactiver cette règle pour cette classe. Le plus simple est de désactiver cette règle pour la classe `DemoApplication` en ajoutant l'annotation **`@SuppressWarnings("PMD.UseUtilityClass")`** juste avant la déclaration de la classe :

Voici la classe `DemoApplication` corrigée :

```java {9}
package be.vinci.ipl.cae.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * Main class of the application.
 */
@SuppressWarnings("PMD.UseUtilityClass")
@SpringBootApplication
public class DemoApplication {

  /**
   * Main method of the application.
   *
   * @param args the arguments
   */
  public static void main(String[] args) {
    SpringApplication.run(DemoApplication.class, args);
  }
}

```

Pour **`JwtAuthenticationFilter.java`**, PMD signale une violation de la règle `LiteralsFirstInComparisons :	Position literals first in String comparisons`. Cela signifie que la classe `JwtAuthenticationFilter` utilise des littéraux dans des comparaisons de chaînes de caractères, ce qui peut entraîner des erreurs si les littéraux sont `null`. Pour corriger cette violation, vous pouvez inverser l'ordre des littéraux et des variables dans les comparaisons de chaînes de caractères :


```java {34} showLineNumbers
@Configuration
public class JwtAuthenticationFilter extends OncePerRequestFilter {

  private final UserService userService;

  /**
   * Constructor for JwtAuthenticationFilter.
   *
   * @param userService the injected UserService.
   */
  public JwtAuthenticationFilter(UserService userService) {
    this.userService = userService;
  }

  /**
   * Filter to handle user authentication.
   *
   * @param request     the request.
   * @param response    the response.
   * @param filterChain the filter chain.
   * @throws ServletException the servlet exception.
   * @throws IOException      the IO exception.
   */
  @Override
  protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
      FilterChain filterChain) throws ServletException, IOException {
    String token = request.getHeader("Authorization");
    if (token != null) {
      String username = userService.verifyJwtToken(token);
      if (username != null) {
        User user = userService.readOneFromUsername(username);
        if (user != null) {
          List<GrantedAuthority> authorities = new ArrayList<>();
          if ("admin".equals(username)) {
            authorities.add(new SimpleGrantedAuthority("ROLE_ADMIN"));
          }
          UsernamePasswordAuthenticationToken authentication =
              new UsernamePasswordAuthenticationToken(
                  user, null, authorities);
          authentication.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
          SecurityContextHolder.getContext().setAuthentication(authentication);
        }
      }
    }
    filterChain.doFilter(request, response);
  }
}
```

Actuellement, notre code ne contient plus de violations de code détectées par PMD 🎉 !

Nous pouvons donc passer à l'étape suivante : la correction des duplications de code détectées par CPD.

N'hésitez pas à relancer les tests pour vérifier que ça ne build pas :

```bash
mvn clean test
```

Pour les erreurs de duplication de code, vous pouvez consulter le rapport HTML des duplications de code généré dans le répertoire **`/target/report/cpd.html`**.

Il y a actuellement de la duplication à la ligne 38 et 63 de la classe `AuthController.java`. Vous pouvez ouvrir le rapport HTML pour voir les détails des duplications et les recommandations pour les corriger.

Voici le code dupliqué dans la classe `AuthController.java` :

```java
public AuthenticatedUser register(@RequestBody Credentials credentials) {
    if (credentials == null
        || credentials.getUsername() == null
        || credentials.getUsername().isBlank()
        || credentials.getPassword() == null
        || credentials.getPassword().isBlank()) {
      throw new ResponseStatusException(HttpStatus.BAD_REQUEST);
    }

    AuthenticatedUser user = userService.register(credentials.getUsername(),
```

Pour corriger les duplications de code, vous pouvez extraire le code dupliqué dans une méthode séparée et appeler cette méthode à partir des endroits où le code dupliqué est utilisé.

Voici comment vous pouvez extraire le code dupliqué dans une méthode séparée nommée `isInvalidCredentials` :

```java {1-7,17-19,38-40} showLineNumbers
  private boolean isInvalidCredentials(Credentials credentials) {
    return credentials == null
        || credentials.getUsername() == null
        || credentials.getUsername().isBlank()
        || credentials.getPassword() == null
        || credentials.getPassword().isBlank();
  }

  /**
   * Register a new user.
   *
   * @param credentials the user credentials from the request body.
   * @return the authenticated user.
   */
  @PostMapping("/register")
  public AuthenticatedUser register(@RequestBody Credentials credentials) {
    if (isInvalidCredentials(credentials)) {
      throw new ResponseStatusException(HttpStatus.BAD_REQUEST);
    }

    AuthenticatedUser user = userService.register(credentials.getUsername(),
        credentials.getPassword());

    if (user == null) {
      throw new ResponseStatusException(HttpStatus.CONFLICT);
    }
    return user;
  }

  /**
   * Login a user.
   *
   * @param credentials the user credentials from the request body
   * @return the authenticated user.
   */
  @PostMapping("/login")
  public AuthenticatedUser login(@RequestBody Credentials credentials) {
    if (isInvalidCredentials(credentials)) {
      throw new ResponseStatusException(HttpStatus.UNAUTHORIZED);
    }

    AuthenticatedUser user = userService.login(credentials.getUsername(),
        credentials.getPassword());

    if (user == null) {
      throw new ResponseStatusException(HttpStatus.UNAUTHORIZED);
    }

    return user;
  }
```

Il est temps de vérifier que tout est en ordre en relançant les tests :

```bash
mvn clean test
```

Vous devriez voir que le build est réussi et qu'il n'y a plus de violations de code détectées par PMD et CPD dans votre projet Java 🚀 !

## Comment gérer les tests unitaires avec Mockito ?

### Introduction

Dans ce tutoriel, toujours dans votre projet `api`, nous allons voir comment nous pouvons aller vers l'automatisation des tests unitaires de notre projet Java en utilisant Maven et le framework de test Mockito.

Nous allons commencer par ajouter les dépendances nécessaires à notre projet Maven pour utiliser JUnit5 & Mockito. Ensuite, nous allons écrire un test unitaire pour une classe de service simple et voir comment nous pouvons simuler les dépendances de cette classe à l'aide de Mockito.

### Ajouter JUnit & Mockito à votre projet Maven

Pour commencer, ajoutez les dépendances de JUnit & Mockito à votre fichier `pom.xml` dans la section `<dependencies>` :

```xml
    <dependency>
      <groupId>org.mockito</groupId>
      <artifactId>junit-jupiter</artifactId>
      <version>2.20.0</version>
    </dependency>
```

### Écrire un test unitaire avec Mockito
Nous allons commencer par tester la classe `PizzaService` de notre projet. Cette classe contient une méthode `readAllPizzas` qui renvoie une Liste de Pizza.

Voici le code de la méthode `readAllPizzas` de la classe `PizzaService` :

```java
 public Iterable<Pizza> readAllPizzas(String order) {
    if (order == null || !order.contains("title")) {
      return pizzaRepository.findAll();
    }

    if (order.contains("-")) {
      return pizzaRepository.findAllByOrderByTitleDesc();
    }
    return pizzaRepository.findAllByOrderByTitleAsc();
  }
``` 

Nous allons écrire un test unitaire pour cette méthode en utilisant Mockito pour simuler le comportement de la dépendance `PizzaRepository`.

Pour ce faire, créez une classe de test `PizzaServiceTest` dans le répertoire `src/test/java` de votre projet :
1. Dans le fichier `PizzaServiceTest.java`, faites un clic droit, puis sélectionnez `Show Context Actions` > `Create test`.
2. Sélectionnez la méthode `readAllPizzas` pour laquelle vous souhaitez créer un test unitaire.
3. Cliquez sur `OK` pour générer la classe de test.
4. Ecrivez un premier scénario qui permet de vérifier que la méthode `readAllPizzas` renvoie bien toutes les pizzas lorsqu'aucun ordre n'est spécifié.

Voici le code de la classe de test `PizzaServiceTest` :

```java
package be.vinci.ipl.cae.demo.services;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.when;

import be.vinci.ipl.cae.demo.models.entities.Pizza;
import be.vinci.ipl.cae.demo.repositories.PizzaRepository;
import java.util.Arrays;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

@ExtendWith(MockitoExtension.class) // Mandatory to use Mockito annotations to initialize mocks
class PizzaServiceTest {

  @Mock
  private PizzaRepository pizzaRepository; // Creates a mock implementation of the specified class or interface.

  @InjectMocks
  private PizzaService pizzaService; // Creates an instance of the class and injects the mocks created with @Mock into it.

  @Test
  void readAllPizzasNoOrderWithNull() {
    // Arrange
    Pizza pizza1 = new Pizza();
    pizza1.setTitle("Margherita");
    Pizza pizza2 = new Pizza();
    pizza2.setTitle("Pepperoni");

    when(pizzaRepository.findAll()).thenReturn(Arrays.asList(pizza1, pizza2));

    // Act
    Iterable<Pizza> result = pizzaService.readAllPizzas(null);

    // Assert
    assertEquals(Arrays.asList(pizza1, pizza2), result);
  }
}
```

Voici une brève explication des annotations utilisées dans la classe de test `PizzaService` :
- **`@ExtendWith(MockitoExtension.class)`** :
    - Cette annotation est nécessaire pour utiliser les annotations Mockito pour initialiser les mocks.
    - Elle permet d'initialiser les mocks avant l'exécution des tests.
    - Si vous oublier cette annotation, vous verrez une erreur `java.lang.NullPointerException` lors de l'exécution des tests.

- **`@Mock`** :
    - Cette annotation crée une implémentation de mock de la classe ou de l'interface spécifiée.
    - Dans notre cas, nous créons un mock de la classe `PizzaRepository` pour simuler son comportement. Ca nous permet de contrôler le comportement de la dépendance `PizzaRepository` dans notre test, sans devoir gérer de DB.
    - Quand vous testez une classe qui dépend d'une autre classe, vous pouvez utiliser des mocks pour simuler le comportement de la classe dépendante (ou des objets "collaborateurs").

- **`@InjectMocks`** :
    - Cette annotation crée une instance de la classe que nous voulons tester et injecte les mocks créés avec `@Mock` dans cette instance.
    - Dans notre cas, nous créons une instance de la classe `PizzaService` et injectons le mock de `PizzaRepository` dans cette instance.
    - Cela nous permet de tester la classe `PizzaService` en utilisant un mock de `PizzaRepository` pour simuler son comportement.

### Exécuter le test unitaire

Vous pouvez bien sûr exécuter le test unitaire en utilisant IntelliJ. Pour cela, il existe plusieurs façons, comme :
- faire un clic droit sur la classe de test `PizzaServiceTest` et sélectionnez `Run 'PizzaServiceTest'`.
- sélectionner la méthode de test `readAllPizzasNoOrderWithNull` et cliquez sur le bouton `Play` à côté de la méthode pour l'exécuter (`Run 'readAllPizzasNoOrd...()'`.

Nous souhaitons automatiser l'exécution des tests unitaires en utilisant Maven. Pour cela, nous allons exécuter les tests unitaires en utilisant la commande suivante au sein d'un terminal d'IntelliJ :

```sh
mvn clean test
```

Vous devriez voir que le test unitaire `readAllPizzasNoOrderWithNull` a été exécuté avec succès et que le test a réussi. Cela signifie que la méthode `readAllPizzas` de la classe `PizzaService` renvoie bien toutes les pizzas lorsqu'aucun ordre n'est spécifié.

Un mini rapport de test au format texte est enregistré dans le répertoire **`/target/surefire-reports`**. Vous pouvez ouvrir ce fichier pour voir les résultats des tests unitaires.

## Comment créer un rapport de couverture de tests ?

Nous allons voir comment générer un rapport de couverture de tests pour notre projet Java en utilisant Maven et le plugin JaCoCo.

JaCoCo est un outil de mesure de la couverture de code pour les applications Java. Il permet de mesurer la couverture de code des tests unitaires et des tests d'intégration, en fournissant des rapports détaillés sur la couverture de code.

Pour commencer, ajoutez le plugin JaCoCo à votre fichier `pom.xml` dans la section `<plugins>` :

```xml
      <plugin>
        <groupId>org.jacoco</groupId>
        <artifactId>jacoco-maven-plugin</artifactId>
        <version>0.8.12</version>
        <executions>
          <execution>
            <goals>
              <goal>prepare-agent</goal>
            </goals>
          </execution>
          <execution>
            <id>report</id>
            <phase>test</phase>
            <goals>
              <goal>report</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
```

Veuillez exécuter la commande suivante au sein d'un terminal d'IntelliJ pour générer un rapport de couverture de tests pour votre projet Java :

```sh
mvn clean test
```

Vous devriez voir un rapport de couverture de tests généré dans le répertoire **`/target/site/jacoco/index.html`**. Vous pouvez ouvrir ce fichier dans un navigateur pour visualiser la couverture de code des tests unitaires dans votre projet Java.

Le rapport de couverture de tests fournit des informations détaillées sur la couverture de code, y compris :
- Le pourcentage de lignes de code couvertes par les tests.
- Le pourcentage de branches de code couvertes par les tests.
- Les classes et les méthodes qui ne sont pas couvertes par les tests.
- Les classes et les méthodes qui sont couvertes par les tests.

Vous pouvez utiliser ce rapport pour identifier les parties du code qui ne sont pas couvertes par les tests unitaires et améliorer la couverture de code de votre projet Java.

Pour notre tutoriel, nous souhaitons atteindre une couverture de code de 100% pour les tests unitaires de la méthode `readAllPizzas` de la classe `PizzaService`. Vous pouvez ajouter des tests supplémentaires pour augmenter la couverture de code et atteindre une couverture de code de 100%.

Veuillez ajouter ces trois tests supplémentaires à `PizzaServiceTest` :

```java
@Test
  void readAllPizzasNoOrderWithWrongOrder() {
    // Arrange
    Pizza pizza1 = new Pizza();
    pizza1.setTitle("Margherita");
    Pizza pizza2 = new Pizza();
    pizza2.setTitle("Pepperoni");

    when(pizzaRepository.findAll()).thenReturn(Arrays.asList(pizza1, pizza2));

    // Act
    Iterable<Pizza> result = pizzaService.readAllPizzas("wrongAurder");

    // Assert
    assertEquals(Arrays.asList(pizza1, pizza2), result);
  }

  @Test
  void readAllPizzasOrderedByTitleAsc() {
    // Arrange
    Pizza pizza1 = new Pizza();
    pizza1.setTitle("Margherita");
    Pizza pizza2 = new Pizza();
    pizza2.setTitle("Pepperoni");
    Pizza pizza3 = new Pizza();
    pizza3.setTitle("Americana");

    when(pizzaRepository.findAllByOrderByTitleAsc()).thenReturn(
        Arrays.asList(pizza3, pizza1, pizza2));

    // Act
    Iterable<Pizza> result = pizzaService.readAllPizzas("title");

    // Assert
    assertEquals(Arrays.asList(pizza3, pizza1, pizza2), result);
  }

  @Test
  void readAllPizzasOrderedByTitleDesc() {
    // Arrange
    Pizza pizza1 = new Pizza();
    pizza1.setTitle("Margherita");
    Pizza pizza2 = new Pizza();
    pizza2.setTitle("Pepperoni");
    Pizza pizza3 = new Pizza();
    pizza3.setTitle("Americana");

    when(pizzaRepository.findAllByOrderByTitleDesc()).thenReturn(
        Arrays.asList(pizza2, pizza1, pizza3));

    // Act
    Iterable<Pizza> result = pizzaService.readAllPizzas("-title");

    // Assert
    assertEquals(Arrays.asList(pizza2, pizza1, pizza3), result);
  }
```

Veuillez exécuter la commande suivante au sein d'un terminal d'IntelliJ pour générer un rapport de couverture de tests pour votre projet Java :

```sh
mvn clean test
```

Vous devriez voir un rapport de couverture de tests généré dans le répertoire **`/target/site/jacoco/index.html`**. Vous pouvez ouvrir ce fichier dans un navigateur pour visualiser la couverture de code des tests unitaires dans votre projet Java. Vous devriez voir que la couverture de code pour les tests unitaires de la méthode `readAllPizzas` de la classe `PizzaService` est de 100%.

Attention : si vous avez un échec dans vos tests, le rapport de couverture de tests ne sera pas généré. Vous devez corriger les erreurs de test pour générer le rapport de couverture de tests.

En cas d'échecs de tests, vous pouvez consulter le rapport de test au format texte dans le répertoire **`/target/surefire-reports`** pour voir les erreurs de test.

Si nécessaire, vous pouvez trouver le code associé aux tutoriels `api` ici : [api](https://github.com/e-vinci/cae-theory-demos/tree/main/api).

---

# 2. Bonnes pratiques et Intégration Continue

## Comment nommer ses Git commits ?

### Introduction

Il existe des conventions pour nommer les commits Git de manière claire et explicite.

Le standard le plus couramment utilisé pour nommer les commits est le **`Conventional Commits`**. Ce standard fournit un ensemble de règles pour créer un historique de commit explicite, ce qui facilite la lecture et la gestion des versions.

Sa spécification est disponible sur le site officiel : https://www.conventionalcommits.org/

**👉 CONSIGNE** : les étudiants Vinci doivent appliquer les règles des **`Conventional Commits`**.

### Format de base

Chaque message de commit doit inclure un **`type`**, une option **`scope`** (portée), et une **`description`** :

```txt
<type>(<scope>): <description>
```

- **type** : le type de la modification (obligatoire).
- **scope** : la portée de la modification (optionnelle). Un scope consiste généralement en un nom de composant ou de module (toute section de votre code) affecté par la modification.
- **description** : une brève description de la modification (obligatoire). Cette description doit commencer par une lettre minuscule et ne doit pas se terminer par un point. En anglais, on utilise souvent l'impératif pour décrire l'action effectuée par le commit (`change` et pas `changed` ni `changes` par exemple).

Pour les **breaking changes** (changements qui cassent la compatibilité avec les versions précédentes), on ajoute un **`!`** après le type et avant la description :

```txt  
<type>(<scope>)!: <description>
```


### Types de commits

Les types de commits les plus couramment utilisés sont :

- **feat** : une nouvelle fonctionnalité.
- **fix** : une correction de bug.
- **docs** : des modifications à la documentation.
- **style** : des modifications qui n'affectent pas le sens du code (espaces, formatage, points-virgules manquants, etc.).
- **refactor** : une modification de code qui n'ajoute pas de fonctionnalité et ne corrige pas de bug.
- **perf** : une modification de code qui améliore les performances. C'est un type spécial de **refactor**
- **test** : ajout ou modification de tests.
- **build** : des modifications qui affectent le système de build, des dépendances externes (npm, yarn, etc.) ou le CI.
- **chore** : des modifications diverses qui n'entrent pas dans les catégories ci-dessus, comme la modification de `.gitignore`.
- **revert** : Un commit qui annule un commit précédent.

### Exemples de commits

Voici quelques exemples de messages de commit conformes aux **`Conventional Commits`** :

```txt
feat: add email notifications on new direct messages
```

```txt
fix(shopping-cart): prevent order an empty shopping cart
```

```txt
feat(api)!: send an email to the customer when a product is shipped
```

```txt
build: update dependencies
```

```txt
refactor: implement fibonacci number calculation as recursion
```

```txt
style: remove empty line
```

### Pourquoi utiliser les **`Conventional Commits`** ?

Les **`Conventional Commits`** permettent :
- d'automatiser la génération de changelogs.
- de communiquer la nature des modifications aux membres d'une équipe, au public ou à des outils.
- de faciliter la lecture et la compréhension de l'historique des commits, ce qui aide les gens lorsqu'ils souhaitent contribuer à un projet.
- d'automatiser le build et le déploiement de projets.


NB : Un changelog est un fichier qui liste toutes les modifications apportées à un projet entre deux versions. Il est généralement utilisé pour informer les utilisateurs des nouvelles fonctionnalités, des corrections de bugs et des changements importants. On peut générer un changelog à partir des messages de commit en utilisant des outils comme `semantic-release` pour le monde "JavaScript" et `Maven` (avec des plugins spécifiques) pour le monde "Java".


## Comment nommer ses Git branches ?

### Introduction

Il n'existe pas de standard universel pour nommer les branches Git, mais il existe des conventions largement adoptées qui aident à maintenir un référentiel organisé et compréhensible.

Il existe une convention de nommage des branches Git appelée **`Conventional Branch`** qui suit les mêmes principes que les **`Conventional Commits`**. Cette convention permet de nommer les branches de manière explicite et structurée.

Sa spécification est disponible sur le site : https://conventional-branch.github.io/

**👉 CONSIGNE** : les étudiants Vinci doivent appliquer les règles de **`Conventional Branch`**.

### Format de base

Chaque nom de branche doit inclure un **`type`** et une **`description`** :

```txt
<type>/<description>
```

### Types de branches

Les types de branches les plus couramment utilisés sont :
- **main** : pour la branche principale du projet. Elle peut aussi être appelée `master` ou `develop`.
- **feature** : pour une nouvelle fonctionnalité.
- **bugfix** : pour une correction de bug.
- **hotfix** : pour des corrections urgentes.
- **release** : pour préparer une nouvelle version.
- **chore** : pour des tâches non liées directement au code, comme la mise à jour de la documentation, mise à jour des dépendances...

### Règles de base

- Les noms de branches doivent être en minuscules et les mots doivent être séparés par des tirets (**`-`**, "hyphens" en anglais).
- Les noms de branches ne doivent pas contenir d'espaces, de caractères spéciaux ou d'accents.
- Les noms de branches doivent être courts et descriptifs.
- Les noms de branches peuvent inclure un identifiant de ticket (par exemple, le numéro d'un ticket Jira ou Trello).

### Exemples de branches

Voici quelques exemples de noms de branches conformes aux **`Conventional Branch`** :

```txt
feature/add-login-page
```

```txt
bugfix/fix-header-bug
```

```txt
release/v1.2.0
```

```txt
chore/update-dependencies
```

```txt
feature/T-123-new-login # Pour un ticket T-123
```

### Pourquoi utiliser les **`Conventional Branch`** ?

Les **`Conventional Branch`** permettent :
- **une communication claire** : le nom d'une branche permet de comprendre rapidement l'objectif de la modification.
- **une automatisation simplifiée** : les outils d'intégration continue peuvent utiliser les noms de branches pour déclencher des actions spécifiques.
- **une organisation d'équipe efficace** : cela encourage la collaboration au sein d'une équipe en rendant l'objectif de chaque branche clair et compréhensible, permettant à chacun de changer de tâches sans confusion.

### Faut-il garder les branches après la fusion ?

Non, il est généralement recommandé de supprimer les branches après les avoir fusionnées à la branche principale.

Pourquoi supprimer les branches après fusion ?
- **Maintenir un référentiel propre** : supprimer les branches fusionnées aide à garder le référentiel propre et organisé. Cela évite l'accumulation de branches obsolètes qui peuvent rendre la navigation dans le référentiel plus difficile.
- **Éviter la confusion** : la présence de nombreuses branches obsolètes peut prêter à confusion pour les membres de l'équipe. Supprimer les branches fusionnées clarifie quelles branches sont actives et pertinentes.
- **Encourager les bonnes pratiques** : cela encourage les bonnes pratiques de gestion des branches, telles que la création de nouvelles branches pour chaque fonctionnalité ou correction de bug.

Quel est l'impact sur l'historique des commits ?
- **l'historique des commits reste intact** : supprimer une branche après fusion n'affecte pas l'historique des commits. Les commits de la branche fusionnée sont intégrés dans la branche principale, et l'historique complet des modifications est conservé.
- **les commits restent accessibles** : les commits de la branche fusionnée restent accessibles dans l'historique de la branche principale. Vous pouvez toujours voir les commits individuels et les modifications apportées.


## Comment créer un projet GitLab  ?

Pour réaliser les tutoriels proposé dans ce cours, nous vous recommandons de créer un projet GitLab personnel. Pour cela, vous pouvez suivre les étapes suivantes :
- Cliquez sur l'onglet `Projects` dans le menu de gauche.
- Cliquez sur `New project` en haut à droite.
- Sélectionnez `Create blank project`.
- Nommez votre projet, par exemple `cae-theory-tutorials`.
- Assurez-vous que le projet est `Private`.
- Cliquez sur `Create project`.

Tous les tutoriels associés à l'intégration continue de code dans GitLab sont à réaliser :
- dans le cadre de votre projet GitLab personnel.
- dans le cadre de votre projet GitLab associé à votre groupe pour les étudiants Vinci.

Pour les étudiants Vinci, il y a donc un ou une membre d'un groupe qui réalisera les tutoriels sur le projet GitLab associé à son groupe, et les autres membres réaliseront les tutoriels sur leur projet GitLab personnel.

## Comment démarrer l'intégration continue de code au sein de GitLab ?

Nous allons dans un premier temps définir ce qu'est l'intégration continue, puis nous verrons comment la mettre en place au sein de GitLab.

### Définition de l'intégration continue

L'intégration continue est une pratique de développement logiciel qui consiste à **automatiser l'intégration du code produit par les développeurs dans un référentiel partagé**. L'objectif est de détecter et de corriger les problèmes d'intégration le plus tôt possible, afin de garantir une qualité de code constante et de faciliter le déploiement continu.

### Mise en place de l'intégration continue avec GitLab

GitLab propose des fonctionnalités intégrées pour mettre en place l'intégration continue de code. Voici les étapes à suivre lorsque l'on souhaite configurer l'intégration continue dans GitLab :
- **Protéger la branche principale** : pour éviter les modifications directes sur la branche principale, il est recommandé de protéger la branche `main` ou `master` et
  d'exiger des *merge requests* pour intégrer les modifications.
- **Créer un fichier `.gitlab-ci.yml`** : ce fichier contient les instructions pour exécuter les tests, le build et le déploiement du code. Il définit les *jobs* et les *stages* à exécuter lors de l'intégration continue.

Au sein de GitLab, les paramètres pour configurer la protection de la branche principale, la gestion des MR et des builds sont accessibles dans les paramètres du projet, ou dans les paramètres du groupe pour appliquer des règles à plusieurs projets.  
Pour les étudiants Vinci, ces paramètres ont été configurés par vos enseignants.
Pour pouvoir configurer ces paramètres, vous devez avoir un rôle de `Maintainer` ou d'`Owner` au niveau du groupe ou projet GitLab.

Lorsque vous créez un projet GitLab, nous vous recommandons de configurer GitLab afin de protéger la branche principale, de configurer les *merge requests* pour qu'elles soient validées par un autre membre de l'équipe avant d'être fusionnées, et de configurer un pipeline d'intégration continue qui exécute les tests unitaires et vérifie la qualité du code.

Pour les étudiants Vinci, concernant le projet GitLab associé à votre groupe :
- vos enseignants ont tenté de paramétrer un maximum de règles au niveau du groupe `2026-cae-projects`, puis d'autres règles plus fines au niveau de votre projet GitLab `cae-group-xy` se trouvant dans le groupe `2026-cae-projects`.
- vous devez avoir créé un compte GitLab en suivant toutes les consignes données ci-dessus.
- vous recevrez un email confirmant que vous avez été ajouté au projet ` e-vinci / cae-projects / 2026-cae-projects / cae-group-xy` avec le rôle de `Maintainer`.
- ce rôle de `Maintainer` vous permettra de créer des *merge requests* et de les fusionner. Vous pouvez également configurer le pipeline d'intégration continue pour exécuter les tests et vérifier la qualité du code.  **👉 CONSIGNE** : attention, avec le rôle de `Maintainer`, vous avez la possibilité de configurer les règles de protection de la branche principale, et les règles de *merge requests*. Ces règles ont été configurées par vos enseignants. Nous vous demandons de ne pas les modifier, sauf demande explicite de vos enseignants.

Pour information, voici les différents rôles disponible dans GitLab :
- **Owner** : peut tout faire, y compris supprimer le groupe ou le projet.
- **Maintainer** : peut tout faire, sauf supprimer le groupe ou le projet. Peut configurer les règles de protection de la branche principale, les règles de *merge requests* et le pipeline d'intégration continue.
- **Developer** : peut créer des *merge requests* et les fusionner, mais ne peut pas configurer les règles de protection de la branche principale ou les règles de *merge requests* au niveau du groupe ; peut mettre en place et exécuter le pipeline d'intégration continue en ajoutant ou modifiant un fichier **`.gitlab-ci.yml`** à la racine d'un projet GitLab ; ne peut pas configurer le CI/CD via les paramètres du projet et ne peut donc pas ajouter des variables d'environnement !
- **Reporter** : peut voir les projets, mais ne peut pas les modifier ; peut créer des issues mais pas des *merge requests* ; ne peut pas pusher du code.
- **Planner** : peut voir les projets, mais ne peut pas les modifier ; peut créer des issues mais pas des *merge requests* ; peut complètement gérer des épics ; ne peut pas pusher du code.
- **Guest** : rôle le plus restrictif, peut seulement voir les projets ; peut créer des issues, mais pas des *merge requests* ; ne peut pas pusher du code.

Pour les détails sur les permissions associées à chaque rôle, vous pouvez consulter la documentation officielle de GitLab : https://docs.gitlab.com/ee/user/permissions.html

Les prochaines sections expliquent, étape par étape, comment configurer l'intégration continue dans GitLab si vous aviez un rôle de `Maintainer` ou d'`Owner` au niveau d'un groupe GitLab ou d'un projet GitLab.

### 🍬 Protection de la branche principale au niveau d'un groupe

Pour les étudiants Vinci, vous n'avez pas de groupe à configurer. Ce paragraphe est donc pour compréhension uniquement du rôle d'un `Maintainer` ou d'un `Owner` au niveau d'un groupe GitLab.

Pour protéger la branche principale au niveau d'un groupe GitLab, vous pouvez suivre les étapes suivantes :
1. Accédez aux paramètres du projet dans GitLab (`Settings`).
2. Cliquez sur `Repository` dans le menu de gauche.
3. Cliquez sur `Default branch`.
4. La branche principale est `main` par défaut (il n'y a rien à changer)
5. Assurez que l'option `Protected` est activée.
6. Cochez les options :
    - Allowed to push : `No one`
    - Allowed to merge : `Developers + Maintainers`.
7. Cliquez sur `Save changes`.

Ainsi, les développeurs ne pourront pas pousser directement sur la branche principale, mais devront passer par des *merge requests*.


### 🍬 Meilleure gestion des MR au niveau d'un groupe

Pour les étudiants Vinci, vous n'avez pas de groupe à configurer. Ce paragraphe est donc pour compréhension uniquement du rôle d'un `Maintainer` ou d'un `Owner` au niveau d'un groupe GitLab.

Vous pouvez configurer les *merge requests* au niveau d'un groupe GitLab pour appliquer des règles à plusieurs projets. Voici comment configurer les *merge requests* au niveau d'un groupe :
1. Accédez aux paramètres du projet dans GitLab (`Settings`).
2. Cliquez sur `General` dans le menu de gauche.
3. Cliquez sur `Merge requests`.
4. Cochez l'option `Pipelines must succeed` : `Merge requests can't be merged if the latest pipeline did not succeed or is still running.`.
5. Cochez l'option `All threads must be resolved`.
6. Cliquez sur `Save changes`.

Ainsi, les *merge requests* ne pourront pas être fusionnées si le pipeline d'intégration continue a échoué ou si tous les threads n'ont pas été résolus.

### Configuration de votre projet sous GitLab

Pour les étudiants Vinci, le projet GitLab associé à votre groupe n'est pas à configurer, car cela a été fait par vos enseignants.

Par contre, pour votre projet GitLab personnel, nous vous recommandons de réaliser ces tâches de configuration pour vous familiariser avec les paramètres de GitLab.

#### Protection de la branche principale

Pour protéger la branche principale, vous ne devez normalement rien changer si vous avez créé votre projet dans un groupe où vous avez déjà protégé la branche principale.

Dans le cadre d'une projet GitLab personnel, pour protéger la branche principale, vous pouvez suivre les étapes suivantes :
1. Accédez aux paramètres du projet dans GitLab (`Settings`).
2. Cliquez sur `Repository` dans le menu de gauche.
3. Cliquez sur `Protected branches`.
4. La branche principale est `main` par défaut (il n'y a rien à changer)
5. Assurez que cela est sélectionné : `Protect this branch`.
    - `Allowed to merge` : `Developers + Maintainers`.
    - `Allowed to push and merge` : `No one`

Ainsi, les développeurs ne pourront pas pousser directement sur la branche principale, mais devront passer par des *merge requests*.

#### Meilleure gestion des MR

##### Historique des commits plus propre

1. Accédez aux paramètres du projet dans GitLab (`Settings`).
2. Cliquez sur `Merge requests` dans le menu de gauche.
3. Cochez l'option `Fast-forward merge`. `Fast-forward merges only.
When there is a merge conflict, the user is given the option to rebase.
If merge trains are enabled, merging is only possible if the branch can be rebased without conflicts.`
4. `Squash commits when merging` : Sélectionnez `Do not allow`. `Squashing is never performed and the checkbox is hidden.`
5. Cliquez sur `Save changes`.

##### Fusion des MR

Si applicable, nous allons configurer les checks qui doivent être validés avant de pouvoir fusionner une *merge request*.

Pour les étudiants Vinci et concernant leur projet GitLab associé à leur groupe, cela a été configuré par vos enseignants.

Concernant votre projet GitLab personnel, si vous avez configuré la gestion des MR au niveau d'un groupe, vous n'avez pas besoin de le refaire au niveau du projet.  
Si vous n'avez pas configuré la gestion des MR au niveau d'un groupe, vous pouvez configurer les *merge requests* au niveau de votre projet GitLab :
1. Accédez aux paramètres du projet dans GitLab (`Settings`).
2. Cliquez sur `Merge requests` dans le menu de gauche.
4. Cochez l'option `Pipelines must succeed` : `Merge requests can't be merged if the latest pipeline did not succeed or is still running.`.
5. Cochez l'option `All threads must be resolved`.
6. Cliquez sur `Save changes`.

##### Approbation des MR

Pour pouvoir configurer les règles d'approbation des *merge requests*, il faut avoir un plan tarifaire payant sur GitLab.com. Il est possible de tester cette fonctionnalité avec un essai gratuit de 60 jours. Néanmoins, pour suivre ce cours théorique, vous n'avez pas besoin de cette fonctionnalité.
Plus d'info sur l'essai possible de GitLab Ultimate : https://about.gitlab.com/free-trial/

Notons que pour les étudiants Vinci, le projet GitLab associé à votre groupe n'est pas à configurer, car cela a été fait par vos enseignants.

Si vous aviez un compte Ultimate, voici comment vous pourriez configurer les règles d'approbation des *merge requests* :
1. Accédez aux paramètres du projet dans GitLab (`Settings`).
2. Cliquez sur `Merge requests` dans le menu de gauche.
3. Veuillez ajouter au moins un membre de votre équipe pour approuver les *merge requests* :
   `Approval required` : `1` (ou plus si vous le souhaitez)
4. Cliquez sur `Save changes`.

## Comment mettre en place le workflow d'intégration continue ?

### Introduction

Voici un premier workflow que nous souhaitons appliquer pour l'intégration continue de code dans GitLab :

1. Un développeur ou une développeuse crée une nouvelle feature branche à partir de la branche principale.
2. Le développeur ou la développeuse travaille sur sa feature et pousse ses modifications sur la branche distante.
3. Le développeur ou la développeuse crée une *merge request* pour fusionner sa branche avec la branche principale.
4. Un autre membre de l'équipe approuve la *merge request*.

Nous allons commencer par ajouter le boilerplate d'une API Spring à votre projet GitLab.


### Clonage de votre projet GitLab personnel

Veuillez cloner votre projet GitLab personnel sur votre machine locale.

Pour cloner votre projet GitLab, vous pouvez utiliser la commande suivante :

```bash
git clone https://gitlab.com/<votre-username-GitLab>/<nom-du-projet>.git
```

S'il y a un souci avec cette commande (par exemple si vous aviez un autre profil GitLab qui serait dans le cache de Git), vous pouvez utiliser la commande suivante :

```bash
git clone https://<votre-username-GitLab>@gitlab.com/<votre-username-GitLab>/<nom-du-projet>.git
```

Vous pouvez utiliser les commandes suivantes pour bien configurer votre compte GitLab sur votre machine locale :

```bash
git config user.name "<votre-username-GitLab>"
git config user.email "<votre-email>@student.vinci.be"
```

### Clonage de votre projet GitLab associé à votre groupe

Cette section est pour les étudiants Vinci uniquement.

**👉 CONSIGNE** : Pour les étudiants Vinci, veuillez cloner votre projet GitLab sur votre machine locale.

> Attention, si vous avez une machine Windows, il est possible que votre Git soit mal configuré, qu'il change les caractères de sauts de ligne de **`lf`** vers **`crlf`** lors du clone du repository. Qu'est-ce que ça signifie ? Les fichiers texte sous Windows utilisent **`crlf`** (carriage return line feed) pour les sauts de ligne, tandis que les fichiers texte sous Unix/Linux utilisent **`lf`** (line feed). Si vous travaillez sur un projet avec des fichiers texte, cela peut poser des problèmes de compatibilité.  
Pour éviter cela, nous vous recommandons d'ajouter la configuration suivante avant de cloner votre projet GitLab :

```bash
git config --global core.autocrlf false
```

Pour vous assurer que vous clonez votre projet GitLab avec votre nouveau compte GitLab, nous vous recommandons cette commande :

```bash
git clone https://<votre-username-GitLab>@GitLab.com/e-vinci/cae-projects/2026-cae-projects/cae-group-<xy>.git
```

En clonant votre projet GitLab avec votre nouveau compte GitLab, vous vous assurez que vous avez les droits nécessaires pour pousser vos modifications sur votre projet GitLab. Git va ajouter une remote `origin` avec votre nouveau compte GitLab.  
NB : une `remote` est un raccourci pour une URL Git. C'est un moyen pratique de ne pas avoir à taper l'URL complète à chaque fois que vous voulez pousser ou tirer des modifications.

Si vous avez plusieurs comptes `GitLab.com` sur votre machine, veuillez vous assurer que les informations d'identification de votre nouveau compte GitLab sont utilisées :

```bash
cd cae-group-<xy>
git config --get user.name
git config --get user.email
```

Si vous avez besoin de changer ces informations, vous pouvez utiliser les commandes suivantes :

```bash
git config user.name "<votre-username-GitLab>"
git config user.email "<votre-email>@student.vinci.be"
```

### Ajout du boilerplate de votre API Spring

Pour rappel, tous les tutoriels associés à l'intégration continue de code dans GitLab sont à réaliser :
- dans le cadre de votre projet GitLab personnel.
- dans le cadre de votre projet GitLab associé à votre groupe pour les étudiants Vinci.

Pour les étudiants Vinci, il y a donc un ou une membre d'un groupe qui réalisera les tutoriels sur le projet GitLab associé à son groupe, et les autres membres réaliseront les tutoriels sur leur projet GitLab personnel.

Vous devriez créer une nouvelle branche à partir de la branche principale pour ajouter le boilerplate de votre API Spring. Nous vous proposons de l'appeler `chore/setup-api-boilerplate`.

NB : Voici un exemple de commandes pour créer une nouvelle branche `chore/setup-api-boilerplate` à partir de la branche principale :

```bash
git checkout main
git pull
git checkout -b chore/setup-api-boilerplate
```

Pour l'ajout du boilerplate de votre API Spring, vous devez avoir fait toutes les étapes qui sont demandés dans la première partie de cette fiche.

Le boilerplate de votre API Spring doit être ajouté à la racine de votre projet GitLab, dans un dossier nommé **`api`**.
Si vous avez suivi les tutoriels, vous devriez avoir un projet Spring Boot avec les dépendances nécessaires pour démarrer votre API. En cas de souci, vous pouvez utiliser le dossier `api` fourni dans ce repository : [cae-project-boilerplate](https://github.com/e-vinci/cae-project-boilerplate).

Pensez à ajouter un fichier `.gitignore` à la racine de votre projet (votre repository) pour ignorer les fichiers inutiles.

```txt
### Mac
.DS_Store
### Windows
Thumbs.db
### Linux
lost+found
.env
*.env
```

Après avoir fait ces modifications, vous devez pousser votre branche `chore/setup-api-boilerplate` (pensez à faire un commit auparavant ; ) sur votre repo distant (qui est votre projet GitLab).

## Comment créer une merge request dans GitLab ?

Sur votre projet GitLab, vous devriez voir un bouton `Create merge request` pour créer une *merge request*.


Veuillez compléter le formulaire de *merge request* en indiquant les informations suivantes :
- **Title** : `Ajout du boilerplate de l'API Spring`
- **Description** : `Ajout du boilerplate de l'API Spring.`
- **Assign to** : `Assign to me`
- **Reviewers** : Si vous êtes dans le cadre d'un projet GitLab associé à un groupe, sélectionnez un membre de votre équipe pour approuver la *merge request*. Pour les étudiants Vinci, vous pouvez sélectionner un autre membre de votre groupe.  Si vous ête dans un projet GitLab personnel, vous pouvez laisser cette option vide.  Dans le cadre d'un projet GitLab personnel, si vous souhaiter jouer le jeu jusqu'au bout, vous pouvez ajouter un nouveau membre à votre projet GitLab (`Manage`, `Members`, `Invite members`). Vous pouvez ensuite mettre à jour votre MR pour ajouter ce nouveau membre en tant que `Reviewer`.
- **Merge options** :
    - `Delete source branch when merge request is accepted.` : Cochez cette option pour supprimer la branche `feature/setup-api-boilerplate` après la fusion. On garde malgré tout l'historique complet des commits.
- Cliquez sur `Create merge request`.


Comme nous n'avons pas encore configuré le pipeline d'intégration continue pour exécuter les tests et vérifier la qualité du code, il n'est pas possible de fusionner la *merge request* pour le moment, même si un autre membre de l'équipe l'approuve.

Vous souhaitez en savoir plus sur comment créer une Merge Request ? [Voir la documentation](https://docs.GitLab.com/ee/user/project/merge_requests/creating_merge_requests.html).

## Comment mettre en place un pipeline ?

### C'est quoi un pipeline ?

Un pipeline est un ensemble de **jobs** qui sont exécutés dans un ordre spécifique pour automatiser le processus de construction, de test et de déploiement d'une application.

Nous allons voir comment configurer un pipeline dans GitLab pour exécuter les tests unitaires et vérifier la qualité du code de notre API.

### Configuration du pipeline

Pour configurer un pipeline dans GitLab, vous devez créer un fichier **`.gitlab-ci.yml`** à la racine de votre projet. Ce fichier contient les instructions pour exécuter les différents jobs du pipeline.

A la racine de votre Projet GitLab (dans votre repository local), dans la branche `chore/setup-api-boilerplate`, créez un fichier **`.gitlab-ci.yml`** avec le contenu suivant :

```yaml
api test:
  image: maven:3.9.9-amazoncorretto-21
  script:
    - cd api
    - mvn clean test
```

Ce fichier définit un job nommé `api test` qui utilise l'image Docker `maven:3.9.9-amazoncorretto-21` pour exécuter les tests unitaires de l'API Spring. Le script exécute les commandes suivantes :
- `cd api` : se déplace dans le dossier `api` où se trouve le code de l'API Spring.
- `mvn clean test` : exécute tous les checks que nous avons configurés dans l'API Spring. Il s'agit du Checkstyle, du PMD, du CPD et des tests unitaires.

Veuillez pousser votre fichier **`.gitlab-ci.yml`** sur votre dépôt distant.

Dans le cadre d'un compte GitLab personnel, il est possible que vous deviez d'abord vérifier votre compte GitLab avant d'avoir le droit d'exécuter une pipeline. Pour cela, veuillez suivre les instructions demandées sur votre projet GitLab.

Pour relancer un build, veuillez simplement pousser un nouveau commit sur votre branche `chore/setup-api-boilerplate`.

Maintenant que vous avez ajouté le fichier **`.gitlab-ci.yml`**, vous devriez voir un pipeline se déclencher automatiquement à chaque *push* sur la branche `chore/setup-api-boilerplate`.

### Vérification du pipeline

Pour vérifier que le pipeline a été exécutée avec succès, vous pouvez accéder à l'onglet `Build`, `Pipelines` de votre projet GitLab.

Là, vous devriez voir un pipeline avec le job `api test` qui a été exécuté avec succès ou qui est toujours en cours.

Si vous cliquez sur le numéro du pipeline, vous pouvez voir les détails de l'exécution du job `api test`. Puis vous pouvez voir les logs de l'exécution du job. On voit ici que tous les tests ont exécutés avec succès et que le build est réussi.

Nous pensons donc que la qualité du commit est bonne et qu'il devrait être possible de fusionner la branche `chore/setup-api-boilerplate` avec la branche principale.

Cependant, il est important de demander à un autre membre de l'équipe de vérifier le code avant de fusionner la branche. Car il est possible que même si la pipeline est réussie, il y ait des erreurs dans le code que seul un humain peut détecter.

## Comment revoir et fusionner une merge request ?

### Revue et approbation de la *merge request*

Veuillez demander à un autre membre de votre équipe, généralement à la personne que vous avez assigné comme `Reviewer` à la création de la *merge request* (MR), de revoir et d'approuver votre MR.

En résumé, pour revoir et approuver une *merge request*, vous devriez :
1. Accéder à la MR.
2. Vérifier que le pipeline a été exécuté avec succès. Si ça n'est pas le cas, demander au créateur de la MR de corriger les erreurs.
3. Regarder les modifications apportées par la MR via l'onglet `Commits` et l'onglet `Changes`. Si les modifications touchent à plusieurs aspects du code, il est recommandé de visualiser localement les modifications. En effet, il est souvent importer de jeter un oeil sur le code pour s'assurer que les modifications sont correctes, mais aussi, d'exécuter l'application pour se rendre compte d'éventuels changements visuels, ou de comportement.

Pour ce faire, nous vous recommandons :
- de cloner la branche de la MR sur votre machine locale.
- de tester les modifications localement.
- il est aussi possible d'installer des extensions à votre éditeur de code pour visualiser localement les Merge Request. Par exemple, pour VS Code, vous pouvez installer l'extension **`GitLab Workflow`**. Dans l'onglet **`ISSUES AND MERGE REQUESTS`**, en faisant un clic droit sur une MR, vous pouvez choisir **`Check out Merge Request Branch`** pour cloner la branche de la MR sur votre machine locale. Vous pouvez ensuite visualiser les modifications apportées par la MR, tester les modifications localement, et ajouter des commentaires sur des lignes spécifiques si nécessaire.

4. Si vous avez des remarques :
- vous pouvez les ajouter dans l'onglet `Changes` sur GitLab (ou directement dans VS Code via l'extension **`GitLab Workflow`**). Pour chaque fichier modifié, vous pouvez ajouter des commentaires sur des lignes spécifiques.
- vous pouvez ajouter des commentaires généraux dans l'onglet `Overview` de la MR.
- vous devez atteindre que le créateur de la MR corrige les erreurs avant d'approuver la MR. Pour cela, le créateur de la MR doit pousser de nouveaux commits sur la branche de la MR.
- vous pouvez revoir les nouveaux commits en cliquant sur le bouton `Commits` de la MR. N'hésitez pas à ajouter des commentaires sur des lignes spécifiques si nécessaire...
5. Approuver la MR en cliquant sur le bouton `Approve` une fois qu'il n'y a plus de commentaires ouverts.

### Fusion de la branche `chore/setup-api-boilerplate` avec la branche principale

Maintenant que le pipeline a été exécutée avec succès et que la MR a été acceptée par le `Reviewer`, vous pouvez fusionner la branche `chore/setup-api-boilerplate` avec la branche principale. Pour cela, il suffit que le créateur de la MR clique sur le bouton `Merge`.

Attention, une fois la MR fusionnée, un nouveau pipeline sera déclenché pour la branche principale. Vous devez vous assurer que ce pipeline est exécuté avec succès avant de continuer.

### Terminer la fusion de la branche `chore/setup-api-boilerplate` avec la branche principale

Tout est bien géré sur le GitLab distant.
Il reste à terminer la fusion de la branche `chore/setup-api-boilerplate` avec la branche principale sur votre machine locale.  
Pour ce faire, vous devez vous placer sur la branche principale et récupérer les dernières modifications. Par exemple, si vous êtes sur la branche `chore/setup-api-boilerplate`, vous pouvez exécuter les commandes suivantes :

```bash
git checkout main
git pull
```

Attention, le git pull n'est possible que si le pipeline de la branche principale est vert.

### Résumé du workflow actuel pour l'intégration continue

Sous forme de résumé visuel, voici le workflow actuel pour l'intégration continue de code dans GitLab :

![Workflow de l'intégration continue](/images/fiche-5-CI-workflow.svg)

Voici le résumé sous forme de texte :
1. Un développeur ou une développeuse crée une nouvelle feature branche à partir de la branche principale.
2. Le développeur ou la développeuse travaille sur sa feature et pousse ses modifications sur la branche distante.
3. Le pipeline d'intégration continue est déclenché automatiquement. Si le pipeline échoue, le développeur ou la développeuse doit corriger les erreurs avant de continuer.
4. Le développeur ou la développeuse crée une *MR* pour fusionner sa branche avec la branche principale et lui assigne un `Reviewer`.
5. Un autre membre de l'équipe, le `Reviewer`, fait une review des modifications, généralement teste localement les modifications, et approuve la *MR* seulement s'il n'a plus de remarques. Tant qu'il y a des remarques, le développeur ou la développeuse (le créateur la créatrice de la MR) doit corriger les erreurs et pousser de nouveaux commits.
6. La *MR* est fusionnée avec la branche principale à la demande du développeur ou de la développeuse.

## Comment gérer des tâches en parallèle dans un pipeline ?

### Introduction

Comment gérer des tâches en parallèle dans un pipeline ? Par exemple, comment exécuter les tests du frontend et de l'API en parallèle dans un pipeline ?

En fait, dans GitLab, vous pouvez définir plusieurs jobs dans un pipeline et les exécuter en parallèle.

Si vous avez plusieurs jobs qui ne dépendent pas les uns des autres, vous pouvez les exécuter en parallèle pour gagner du temps.

Nous allons donc ajouter notre frontend React dans le pipeline et exécuter les tests du frontend et de l'API en parallèle.

### Ajout du boilerplate de votre frontend

Pour l'ajout du boilerplate de votre frontend à votre projet GitLab, vous devez avoir fait tout ce qui est demandé dans la fiche précédente ([Fiche 4](../fiche-4/)).
A la fin de la fiche 4 (le dernier projet s'appelle `unit-tests`), vous devriez avoir un projet React avec les dépendances nécessaires pour démarrer un frontend. Néanmoins, ce frontend n'intègre pas la librairie MUI.

Dès lors, pour vous aider, nous avons créé un boilerplate pour le frontend qui reprend les tutoriels précédents et qui intègre la librairie MUI. Vous pouvez le trouver ici : [frontend du cae-project-boilerplate](https://github.com/e-vinci/cae-project-boilerplate/tree/main/frontend).

Vous devriez créer une nouvelle branche à partir de la branche principale pour ajouter le boilerplate de votre frontend React dans un dossier nommé **`/frontend`**. Nous vous proposons d'appeler votre nouvelle branche `chore/setup-frontend-boilerplate`.

### Mise à jour du pipeline

Veuillez mettre à jour votre fichier `.gitlab-ci.yml` pour exécuter les tests du frontend et de l'API en parallèle :

```yaml {7-13}
api test:
  image: maven:3.9.9-amazoncorretto-21
  script:
    - cd api
    - mvn clean test

frontend test:
  image: node:20
  script:
    - cd frontend
    - npm install
    - npm run lint
    - npm run coverage
```

Ce fichier définit deux jobs `api test` et `frontend test` qui exécutent les tests de l'API Spring et du frontend React en parallèle.

Ici, pour le frontend, nous souhaitons d'abord exécuter les tests du linter, puis les tests unitaires pour générer un rapport de couverture de code.

Veuillez pousser tous vos modifications sur votre repo distant.

### Vérification du pipeline

Après avoir poussé vos modifications, vous devriez voir un nouveau pipeline se déclencher automatiquement à chaque *push* sur la branche `chore/setup-frontend-boilerplate`.

Vous devriez voir deux jobs `api test` et `frontend test` qui sont exécutés en
parallèle.

Une fois le pipeline exécuté avec succès, nous pourrions fusionner la branche `chore/setup-frontend-boilerplate` avec la branche principale.

Mais avant cela, nous souhaitons découvrir un nouveau concept, la notion d'artefacts pour stocker les rapports de tests générés par les jobs du pipeline.

## Comment sauvegarder des rapports dans GitLab ?

### Introduction

Dans un pipeline, il est souvent nécessaire de générer des rapports de tests, de couverture de code, de qualité du code, etc. Ces rapports peuvent être stockés dans GitLab en tant qu'artefacts pour les consulter ultérieurement.

Nous allons voir comment sauvegarder les rapports de tests générés par les jobs `api test` et `frontend test` dans GitLab.

### Sauvegarde des rapports

Pour sauvegarder les rapports de tests dans GitLab, vous devez utiliser l'option `artifacts` dans le fichier `.gitlab-ci.yml`.

Voici comment mettre à jour votre fichier `.gitlab-ci.yml` pour sauvegarder les rapports de tests :

```yaml {6-10,19-21} showLineNumbers
api test:
  image: maven:3.9.9-amazoncorretto-21
  script:
    - cd api
    - mvn clean test
  artifacts:
    paths:
      - api/target/reports/
      - api/target/site/
      - api/target/surefire-reports/

frontend test:
  image: node:20
  script:
    - cd frontend
    - npm install
    - npm run lint
    - npm run coverage
  artifacts:
    paths:
      - frontend/coverage/
```

Ce fichier définit deux jobs `api test` et `frontend test` qui exécutent les tests de l'API Spring et du frontend React en parallèle. Les rapports de tests générés par ces jobs sont sauvegardés dans GitLab en tant qu'artefacts.

Pour l'API Spring, nous sauvegardons les rapports dans les dossiers `api/target/reports/`, `api/target/site/` et `api/target/surefire-reports/`. Pour le frontend React, nous sauvegardons les rapports dans le dossier `frontend/coverage/`.

Pour retrouver ces documents dans GitLab, vous pouvez accéder à l'onglet `Build`, `Pipelines` de votre projet GitLab. Vous devriez voir un pipeline avec les jobs `api test` et `frontend test` qui ont été exécutés avec succès.

Si vous cliquez sur le numéro du pipeline, vous pouvez voir les détails de l'exécution du job `api test` ou `frontend test`.
En cliquant sur le job, vous pouvez télécharger ou voir les artefacts générés par le job.

Pour voir les artefacts générés par le job `api test`, vous pouvez cliquer sur le job `api test` dans le pipeline. Cliquez ensuite sur le bouton `Browse` pour voir les artefacts générés par le job.

Idem pour le job `frontend test`. Allez dans le pipeline, cliquez sur le job `frontend test` et cliquez sur le bouton `Browse` pour voir les artefacts générés par le job.

### Intégration des changements dans la branche principale

Maintenant que vous avez ajouté les artefacts dans votre pipeline, vous pouvez fusionner la branche `chore/setup-frontend-boilerplate` avec la branche principale.

Pour cela, vous devez créer une *merge request* pour fusionner la branche `chore/setup-frontend-boilerplate` avec la branche principale. Une fois la *merge request* approuvée, vous pouvez cliquer sur le bouton `Merge` pour fusionner la branche `chore/setup-frontend-boilerplate` avec la branche principale.

N'oubliez pas de mettre à jour votre branche principale sur votre machine locale après la fusion.

## Est-ce que l'intégration continue est vraiment utile ?

### Intro

Imaginons que quelqu'un vient à changer le code du frontend ou de l'API. Il est possible que ces changements affectent le bon fonctionnement de l'application.

L'intégration continue permet de détecter ces problèmes le plus tôt possible, en exécutant automatiquement les tests et en vérifiant la qualité du code à chaque modification du code.

Pour ce tutoriel, veuillez créer un souci au niveau du frontend.

### Création d'un souci au niveau du frontend

Vous savez qu'il est impossible de faire un `push` directement sur la branche principale. Vous devez passer par une *merge request*.

Veuillez donc créer une nouvelle branche au niveau du frontend que nous allons appeler `feature/update-login-page`.

Veuillez mettre à jour le fichier `src/components/pages/LoginPage.tsx` pour changer la couleur du formulaire de connexion (de 'secondary.light' vers "primary.light").

{% raw %}
```tsx {6}
return (
    <Box
      sx={{
        margin: 2,
        padding: 3,
        backgroundColor: "primary.light",
        borderRadius: 4,
        boxShadow: 2,
      }}
    >
```
{% endraw %}
Faite un `commit` et un `push` sur votre branche `feature/update-login-page`.

### Vérification du pipeline

Après avoir poussé vos modifications, vous devriez voir un nouveau pipeline se déclencher automatiquement à chaque *push* sur la branche `feature/update-login-page`.

Vous devriez voir deux jobs `api test` et `frontend test` qui sont exécutés en parallèle.

Et finalement, vous devriez voir que le job `frontend test` a échoué. En cliquant sur le job `frontend test`, vous pouvez voir les détails de l'échec.
Voici le message d'erreur que vous devriez voir :

```txt
eslint . --ext ts,tsx --report-unused-disable-directives --max-warnings 0
/builds/e-vinci/cae-projects/2026-cae-projects/cae-xy/frontend/src/components/pages/LoginPage.tsx
  40:26  error  Replace `"primary.light"` with `'primary.light'`  prettier/prettier
✖ 1 problem (1 error, 0 warnings)
  1 error and 0 warnings potentially fixable with the `--fix` option.
```

On voit ici que le linter ESLint a détecté une erreur dans le fichier `src/components/pages/LoginPage.tsx`. Nous avons utilisé des guillemets doubles au lieu de simples pour la couleur du formulaire de connexion !!!

### Correction de l'erreur

Pour corriger cette erreur, vous devez mettre à jour le fichier `src/components/pages/LoginPage.tsx` pour changer la couleur du formulaire de connexion de `"primary.light"` vers `'primary.light'`.

{% raw %}
```tsx {6}
return (
    <Box
      sx={{
        margin: 2,
        padding: 3,
        backgroundColor: 'primary.light',
        borderRadius: 4,
        boxShadow: 2,
      }}
    >
```
{% endraw %}

Faite un `commit` et un `push` sur votre branche `feature/update-login-page`.

Cette fois-ci, le job `frontend test` devrait être exécuté avec succès ! 🎉

Si nécessaire, vous pouvez trouver le code associé à l'intégration continue ici : [ici](https://github.com/e-vinci/cae-theory-demos/tree/main/ci).
 