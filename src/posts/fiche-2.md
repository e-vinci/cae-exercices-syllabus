---
title: Fiche 2
description: Spring et bases de données.
permalink: posts/{{ title | slug }}/index.html
date: '2026-02-12'
tags: [fiche, spring, orm]
---

# Spring & JPA — Plateforme de cours

## Pourquoi cette fiche ?

À la fiche précédente nous avons créé une mini API Spring: contrôleurs, services, injection de dépendances, fichiers `.http` pour tester. Cette fois nous allons connecter notre application à une base de données relationnelle **PostgreSQL** pour construire une plateforme universitaire : gestion des cours, leçons, étudiants inscrits et syllabus détaillés. L’objectif est double :

1. Comprendre comment Spring Data JPA (l’ORM) transforme nos classes Java en tables SQL et inversement.
2. Apprendre à modéliser différentes relations (One-to-Many, Many-to-Many, One-to-One) tout en validant et exposant nos données via une API REST.

### Objectifs pédagogiques

- Installer/configurer PostgreSQL localement (service natif ou Docker) et brancher Spring dessus.
- Revoir les notions d’ORM, d’annotations JPA et la raison d’être des `Repository`.
- Implémenter un CRUD complet pour l’entité `Course` en utilisant `@RequestBody`, `@ResponseStatus`, `@Valid`, `ResponseStatusException`.
- Illustrer trois relations différentes :
  - `Course` → `Lesson` (One-to-Many)
  - `Course` ↔ `Student` (Many-to-Many)
  - `Course` ↔ `CourseDetail` (One-to-One avec `@MapsId`)
- Introduire de bonnes pratiques de validation et de tests HTTP.

> **Question d’amorçage :** En quoi la séparation Controller → Service → Repository rend-elle votre application plus facile à tester ? Prenez 2 minutes pour y répondre avant de continuer.

---

## Partie 1 — Rappels conceptuels

### 1.1 ORM, JDBC, JPA, Hibernate

| Terme | Rôle |
| --- | --- |
| JDBC | API bas niveau pour exécuter des requêtes SQL depuis Java. |
| JPA  | Spécification décrivant comment mapper des objets Java vers des tables relationnelles. |
| Hibernate | Implémentation la plus utilisée de JPA (celle que Spring embarque par défaut). |

Concrètement : votre entité `Course` devient une table `courses`; un objet `new Course(...)` devient une ligne; et `courseRepository.findAll()` produit un `SELECT * FROM courses`. C’est l’ORM (Object-Relational Mapper) qui effectue ces conversions.

> **À méditer :** Dans quel cas **refuseriez-vous** d’utiliser un ORM (ex: requêtes ultra-optimisées, contrôle complet du SQL) ?

### 1.2 Architecture en couches

1. **Controller** — Dialogue HTTP: paramètres, body JSON, codes réponse.
2. **Service** — Règles métier : vérifier une capacité maximale de cours, transformer un DTO, etc.
3. **Repository** — Accès aux données (SQL généré par Hibernate).
4. **Base** — Postgres garde la vérité persistante.

Chaque couche parle son vocabulaire : HTTP pour le contrôleur, Java métier pour le service, SQL pour le repository. Cette séparation permet d’écrire des tests ciblés et de faire évoluer les couches indépendamment.

---

## Partie 2 — Initialiser le projet Spring Boot

Créez un nouveau projet Spring boot (`fiche2`) avec les dépendances suivantes :
- `Spring Web` - pour construire une API REST.
- `Spring Data JPA` - pour l’ORM et les repositories.
- `PostgreSQL Driver` - pour connecter Spring à Postgres.
- `Spring Boot Starter Validation` - pour activer les annotations de validation.
---

## Partie 3 — Préparer l’environnement

### Option 1 : Installer Postgres localement

Cette option part du principe que vous avez déjà installé PostgreSQL 16 (ou une version récente) sur votre machine.

1. Démarrer le service Postgres.
   - macOS : `brew services start postgresql@16`
   - Linux : `sudo service postgresql start`
   - Windows : via *Services* (`services.msc`) ou PowerShell admin `net start postgresql-x64-16` / `pg_ctl -D "C:\\Program Files\\PostgreSQL\\16\\data" start`
2. Se connecter avec un utilisateur admin :
   ```bash
   psql -U postgres
   ```
3. Créer base + utilisateur :
   ```sql
   CREATE DATABASE cae_db;
   CREATE USER cae_user WITH ENCRYPTED PASSWORD 'cae';
   GRANT ALL PRIVILEGES ON DATABASE cae_db TO cae_user;
   \c cae_db;
   GRANT ALL ON SCHEMA public TO cae_user;
   ```
4. Vérifier l’accès :
   ```bash
   psql -h localhost -U cae_user -d cae_db -W
   ```

### Option 2 : Docker

Plutôt que d’installer PostgreSQL localement, il est possible d’utiliser Docker pour lancer une instance isolée dans un conteneur.

Plus tard dans le projet, il sera nécessaire d'utiliser Docker. Vous pouvez donc choisir cette option dès maintenant pour éviter d’avoir à faire les deux configurations.

Commencez par installer l'application Docker Desktop (Windows/macOS), ou les paquets `docker` et `docker-compose-plugin` (Linux). Vous pouvez vérifier que tout est en place avec `docker --version` et `docker compose version`.

Ensuite, créez un fichier `docker-compose.yml` à la racine du projet avec le contenu suivant :

```yaml
services:
  db:
    image: postgres:latest
    container_name: cae_exercices_fiche2_db
    environment:
      POSTGRES_DB: cae_db
      POSTGRES_USER: cae_user
      POSTGRES_PASSWORD: cae
    ports:
      - "5433:5432"
```

Vous pouvez lancer la base de données avec la commande `docker compose up -d`, ou directement dans IntelliJ en ouvrant le fichier et en cliquant sur le bouton de lancement à gauche de la ligne `db:`. 

Le mapping `5433:5432` signifie que vous accéderez à Postgres sur le port `5433` de votre machine, tandis que le conteneur lui-même écoute sur le port standard `5432`. Nous avons ici choisi `5433` pour éviter un conflit avec une éventuelle instance locale de Postgres déjà présente sur `5432`. Si vous n’avez pas d’autre instance, vous pouvez aussi utiliser `5432:5432` pour garder le port standard.

### Liaison du projet à la base de données

Dans le fichier `application.properties`, ajoutez la configuration suivante pour que Spring puisse se connecter à votre base Postgres :

```properties
spring.datasource.driver-class-name=org.postgresql.Driver
spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect

spring.datasource.url=jdbc:postgresql://localhost:5433/cae_db
spring.datasource.username=cae_user
spring.datasource.password=cae

spring.jpa.hibernate.ddl-auto=create
spring.jpa.generate-ddl=true
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```

Le premier bloc indique à Spring d’utiliser le driver Postgres et le dialecte SQL adapté. Le second bloc configure la connexion (URL, utilisateur, mot de passe). Le troisième bloc active la génération automatique du schéma (création des tables) à partir de nos entités Java.

> ⚠️ Pensez à modifier le port dans l’URL si vous avez choisi un mapping différent dans Docker ou si vous utilisez une instance locale sur un autre port.

Le mode `ddl-auto` permet de (re)créer les tables à chaque démarrage, ce qui est pratique pour cette fiche. Cependant, en production, il est recommandé de gérer la base de données séparément du projet (via des migrations Flyway ou Liquibase) et de désactiver `ddl-auto` pour éviter les modifications imprévues.

`ddl-auto` options :
| Valeur | Effet | Usage |
| --- | --- | --- |
| create | Supprime + recrée les tables à chaque démarrage. | Démos rapides. |
| create-drop | Idem, mais supprime les tables à l’arrêt. | Exercices/tests manuels. |
| update | Essaie d’adapter le schéma existant. | POC local (attention aux surprises). |
| validate | Vérifie que le schéma correspond aux entités, ne change rien. | Environnements stables. |
| none | Hibernate n’intervient pas. | Production avec migrations Flyway/Liquibase. |

> **Question :** Pourquoi `update` n’est-il pas conseillé en prod ? (Indice : suppression/renommage de colonnes).


---

## Partie 4 — Première version sans base de données

Le contexte de cette fiche est de construire une plateforme de gestion de cours universitaires. Un cours a un titre, une description et un nombre de crédits. Il est composé de plusieurs leçons. Les étudiants peuvent s’inscrire à plusieurs cours, et chaque cours peut avoir plusieurs étudiants. Enfin, chaque cours peut avoir un détail optionnel (syllabus, quota max, contact).

Avant de créer sa table en base de données, mettons en place l'entité `Course` avec une gestion simple en mémoire.

### 4.1 Modèle `Course`
```java
public class Course {
    // Constructeur vide requis par JPA / Jackson
    public Course() {}

    public Course(String title, String description, int credits) {
        this.title = title;
        this.description = description;
        this.credits = credits;
    }

    private String title;
    private String description;
    private int credits;

    // getters / setters générés par l'IDE
}
```

L'utilisation de JPA implique certaines contraintes sur la structure de vos classes :
- Constructeur vide → JPA et Jackson instancient l’objet vide avant de remplir les champs.
- Attributs privés + getters/setters → Hibernate utilise les getters/setters pour accéder aux propriétés et remplir les champs.

### 4.2 Service/Controller
Service qui renvoie une liste fixe de cours; controller exposant `/courses/`.

```java
@Service
public class CoursesService {
    public Iterable<Course> getAllCourses() {
        return List.of(
            new Course("Spring Boot 101", "Introduction", 5),
            new Course("Bases de données", "Modèle relationnel", 4)
        );
    }
}

@RestController
@RequestMapping("/courses")
public class CoursesController {
    private final CoursesService service;

    public CoursesController(CoursesService service) {
        this.service = service;
    }

    @GetMapping("/")
    public Iterable<Course> getCourses() {
        return service.getAllCourses();
    }
}

```
Tester avec un fichier `.http` :
```http
GET http://localhost:8080/courses/
```

Nous sommes prêts à brancher la base de données.

---

## Partie 5 — Entités & Repositories (avec Postgres)

### 5.1 Transformer `Course` en entité JPA

Nous allons enrichir la classe `Course` avec les annotations JPA pour qu’elle soit automatiquement mappée à une table en base de données. Nous ajoutons aussi un champ `id` qui servira de clé primaire auto-incrémentée.

```java
@Entity
@Table(name = "courses")
public class Course {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(unique = true)
    @NotBlank
    @Size(max = 100)
    private String title;

    @NotBlank
    @Size(max = 500)
    private String description;

    @Positive
    private int credits;

    public Course() {}
    public Course(String title, String description, int credits) {
        this.title = title;
        this.description = description;
        this.credits = credits;
    }
    // getters/setters
}
```

- `@Entity` indique que cette classe correspond à une table en base de données.
- `@Table(name = "courses")` précise le nom de la table (sinon ce serait `course` par défaut).
- `@Id` + `@GeneratedValue` créent la clé primaire et gèrent son auto-incrémentation.
- `@Column(unique = true)` force une contrainte SQL
- `@Size`, `@NotBlank`, `@Positive` sont des annotations de validation qui seront vérifiées par le controlleur lors de la création ou mise à jour d’un cours.

### 5.2 Repository

Nous ajoutons maintenant une nouvelle couche **Repository** qui va gérer l’accès aux données. Chaque fonction permet d'effectuer directement une action dans la table de base de données correspondante. 

Spring Data JPA fournit des interfaces prêtes à l’emploi pour les opérations CRUD de base, et permet aussi de définir des méthodes personnalisées basées sur le nom.

```java
@Repository
public interface CoursesRepository extends CrudRepository<Course, Long> {
    Optional<Course> findByTitle(String title);
}
```

Spring Data génère l’implémentation de cette interface au démarrage de l'application. Cette implémentation contiendra le code SQL correspondant à l'action recherchée. 

L'interface étendue `CrudRepository` fournit des opérations standard : `save`, `findAll`, `findById`, `deleteById`, etc. Elle utilise les types génériques pour générer les signatures de ces méthodes selon le type de l'entité `Course` et sa clé primaire `Long`.

Nous ajoutons également une méthode personnalisée `findByTitle`. Spring Data analysera le nom de la méthode et générera automatiquement la requête SQL correspondante (`SELECT * FROM courses WHERE title = ?`). Il est possible de créer des méthodes plus complexes (ex: `findByCreditsGreaterThan(int credits)`) en respectant des conventions de nommage. Vous pouvez retrouver la liste complète des mots-clés dans la documentation officielle de Spring Data JPA : [JPA Query methods](https://docs.spring.io/spring-data/jpa/reference/jpa/query-methods.html)

> **Question :** Que se passe-t-il si vous faites une faute de syntaxe en créant une query personnalisée, par exemple `findByTitel` ? Testez le.

### 5.3 `CommandLineRunner` pour précharger des données

Une méthode annotée avec `@Bean` et retournant un `CommandLineRunner` est exécutée au démarrage de l’application. C’est un bon endroit pour précharger des données de test dans la base de données.

```java
@SpringBootApplication
public class UniversityApplication {

    ...

    @Bean
    CommandLineRunner seed(CoursesRepository coursesRepository) {
        return args -> {
            coursesRepository.save(new Course("Spring Boot 101", "Introduction", 5));
            coursesRepository.save(new Course("Bases de données", "Relationnel & SQL", 4));
            coursesRepository.save(new Course("DevOps", "CI/CD", 3));
        };
    }
}
```

Relancez l’app et vérifiez dans votre client SQL que la table `courses` contient ces entrées. Observez les logs (affichés grâce à `spring.jpa.show-sql=true`).

---

## Partie 6 — Service & Controller complets

Nous complétons notre service et notre contrôleur pour implémenter un CRUD complet : création, lecture, mise à jour et suppression de cours.

### 6.1 Service

```java
@Service
public class CoursesService {
    private final CoursesRepository coursesRepository;

    public CoursesService(CoursesRepository coursesRepository) {
        this.coursesRepository = coursesRepository;
    }

    public Iterable<Course> getAllCourses() {
        return coursesRepository.findAll();
    }

    public Optional<Course> getCourse(long id) {
        return coursesRepository.findById(id);
    }

    public Course createCourse(Course course) {
        if (coursesRepository.findByTitle(course.getTitle()).isPresent()) {
            return null;
        }
        return coursesRepository.save(course);
    }

    public Course updateCourse(long id, CourseUpdateDTO payload) {
        Course existing = coursesRepository.findById(id).orElseNull();
        if (existing == null) {
            return null;
        }

        if (payload.getTitle() != null) existing.setTitle(payload.getTitle());
        if (payload.getDescription() != null) existing.setDescription(payload.getDescription());
        if (payload.getCredits() != null) existing.setCredits(payload.getCredits());

        return coursesRepository.save(existing);
    }

    public void deleteCourse(long id) {
        coursesRepository.deleteById(id);
    }
}
```

La classe `CourseUpdateDTO` est un simple DTO (Data Transfer Object) qui contient les champs modifiables d’un cours. Il permet de faire des mises à jour partielles sans exposer l’entité complète. Nous pouvons utiliser un record Java pour cela, qui génère automatiquement les getters et le constructeur. 

Les records ne sont pas compatibles avec JPA parce qu'ils ne possèdent pas de constructeur sans arguments. Ils ne peuvent donc pas être utilisés comme entités. Mais comme ce DTO n’est utilisé que pour la communication entre le client et le service, cela ne pose pas de problème.

```java
public record CourseUpdateDTO(
    @NotBlank @Size(max = 100) String title, 
    @NotBlank @Size(max = 500) String description, 
    @Positive Integer credits
) {}
```

### 6.2 Controller

```java
@RestController
@RequestMapping("/courses")
public class CoursesController {
    private final CoursesService coursesService;

    public CoursesController(CoursesService coursesService) {
        this.coursesService = coursesService;
    }

    @GetMapping("/")
    public Iterable<Course> listCourses() {
        return coursesService.getAllCourses();
    }

    @GetMapping("/{id}")
    public Course getCourse(@PathVariable long id) {
        return coursesService.getCourse(id)
            .orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND));
    }

    @PostMapping("/")
    @ResponseStatus(HttpStatus.CREATED)
    public Course createCourse(@Valid @RequestBody Course course) {
        Course created = coursesService.createCourse(course);
        if (created == null) {
            throw new ResponseStatusException(HttpStatus.CONFLICT);
        }
        return created;
    }

    @PutMapping("/{id}")
    public Course updateCourse(@PathVariable long id, @Valid @RequestBody CourseUpdateDTO payload) {
        Course updated = coursesService.updateCourse(id, payload);
        if (updated == null) {
            throw new ResponseStatusException(HttpStatus.NOT_FOUND);
        }
        return updated;
    }

    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void deleteCourse(@PathVariable long id) {
        if (coursesService.getCourse(id).isEmpty()) {
            throw new ResponseStatusException(HttpStatus.NOT_FOUND);
        }
        coursesService.deleteCourse(id);
    }
}
```

Explications :
- Les annotations `@ResponseStatus` sur les méthodes indiquent le code HTTP à renvoyer en cas de succès. 
- En cas d’erreur (ex: cours non trouvé), nous lançons une `ResponseStatusException` qui permet de renvoyer un code d’erreur précis.
- L'annotation `@PathVariable` indique que la valeur du paramètre doit être extraite de l'URL. Le segment `{id}` dans l'URL correspond au paramètre `long id` de la méthode, le nom doit correspondre pour que Spring puisse faire le lien.
- L'annotation `@RequestBody` indique que le corps de la requête HTTP doit être désérialisé en objet avec Jackson. 
- L'annotation `@Valid` déclenche la validation des champs de l'objet selon les annotations de validation présentes dans la classe (`@NotBlank`, `@Size`, `@Positive`). Si la validation échoue, Spring renvoie automatiquement un `400 Bad Request` avec les détails des erreurs. Certaines validations plus complexes (ex: la description doit être plus longue que le titre) nécessitent une validation personnalisée qui peuvent être implémentées directement dans le contrôleur.

---

## Partie 7 — Tests HTTP

Créer `tests/courses.http` :
```http
@baseUrl = http://localhost:8080

### Lister
GET {{baseUrl}}/courses/

### Créer
POST {{baseUrl}}/courses/
Content-Type: application/json

{
  "title": "Architecture logicielle",
  "description": "Patterns & DDD",
  "credits": 6
}

### Lire
GET {{baseUrl}}/courses/1

### Mettre à jour
PUT {{baseUrl}}/courses/1
Content-Type: application/json

{
  "description": "Patterns avancés",
  "credits": 7
}

### Supprimer
DELETE {{baseUrl}}/courses/1

### Vérifier 404
GET {{baseUrl}}/courses/1
```

---

## Partie 8 — Relations avancées

Nous enrichissons le modèle avec trois entités supplémentaires.

### 8.1 One-to-Many : `Course` → `Lesson`

- **Réalité :** un cours possède plusieurs leçons planifiées.
- **Modélisation :** relation One-to-Many côté `Course`, clé étrangère `course_id` côté `Lesson`.

```java
@Entity
@Table(name = "lessons")
public class Lesson {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String title;
    private LocalDateTime scheduledAt;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "course_id")
    @JsonBackReference
    private Course course;
}

@Entity
@Table(name = "courses")
public class Course {
    ...
    @OneToMany(mappedBy = "course", cascade = CascadeType.ALL, orphanRemoval = true)
    @JsonManagedReference
    private List<Lesson> lessons = new ArrayList<>();
}
```
Explications :
- Commencez par la direction `Lesson -> Course` uniquement si vous n’avez pas besoin de renvoyer les leçons depuis `/courses`. Ajoutez la liste plus tard pour exposer un cours avec toutes ses leçons.
- `@JoinColumn(name = "course_id")` crée une colonne `course_id` dans la table `lessons` qui référence la clé primaire de `courses`. L'attribut `course` dans `Lesson` est la référence à l'entité `Course` associée à cette leçon par cette clé étrangère mais il ne correspond pas à une colonne en tant que tel.
- `mappedBy = "course"` indique que c’est la classe `Lesson` qui possède la relation (via le champ `course`), et que la table `courses` n’a pas de colonne supplémentaire pour gérer cette relation.
- `fetch = FetchType.LAZY` évite de charger les leçons à chaque fois que vous récupérez un cours (optimisation).
- `@JsonManagedReference/@JsonBackReference` évitent la récursion infinie lors de la sérialisation JSON. `@JsonManagedReference` indique que c’est le côté parent (Course) qui gère la relation, tandis que `@JsonBackReference` indique que c’est le côté enfant (Lesson) qui ne doit pas être sérialisé pour éviter la boucle.
- `cascade = CascadeType.ALL` permet de persister/supprimer les leçons automatiquement avec le cours. `orphanRemoval = true` supprime les leçons orphelines (celles qui ne sont plus associées à un cours).

### 8.2 Many-to-Many : `Course` ↔ `Student`

- **Réalité :** un étudiant suit plusieurs cours, et un cours contient plusieurs étudiants.
- **Modèle :** table de jointure `enrollments` générée automatiquement.

```java
@Entity
@Table(name = "students")
public class Student {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String firstName;
    private String lastName;

    @ManyToMany(mappedBy = "students")
    @JsonIgnore
    private Set<Course> courses = new HashSet<>();
}

@Entity
public class Course {
    ...
    @ManyToMany
    @JoinTable(
        name = "enrollments",
        joinColumns = @JoinColumn(name = "course_id"),
        inverseJoinColumns = @JoinColumn(name = "student_id")
    )
    private Set<Student> students = new HashSet<>();
}
```

Explications :
- Utilise un `Set` au lieu d'une `List` pour éviter les doublons.
- `mappedBy = "students"` indique que c’est la classe `Student` qui possède la relation (via le champ `courses`), et que la table `courses` n’a pas de colonne supplémentaire pour gérer cette relation.
- `@JoinTable` définit la table de jointure `enrollments` qui contient les associations entre `course_id` et `student_id`. Cette table est gérée automatiquement par JPA.
- `JsonIgnore` sur le côté `Student` évite la récursion infinie lors de la sérialisation JSON. Si vous souhaitez exposer les étudiants d’un cours, vous pouvez laisser `@JsonManagedReference` sur le côté `Course` et `@JsonBackReference` sur le côté `Student` à la place.
- Si on souhaite stocker la date d’inscription ou la note, il est possible de transformer la table de jointure en entité propre `Enrollment` avec sa propre clé.
- Pensez à mettre à jour les deux côtés (`course.getStudents().add(student); student.getCourses().add(course);`).

### 8.3 One-to-One : `Course` ↔ `CourseDetail`

`CourseDetail` stocke des informations optionnelles (syllabus PDF, quota max, contact). Il partage la même clé primaire que `Course`.

```java
@Entity
@Table(name = "course_details")
public class CourseDetail {
    @Id
    private Long id;

    @OneToOne
    @MapsId
    @JoinColumn(name = "course_id")
    private Course course;

    private String syllabusUrl;
    private Integer maxStudents;
}
```

Explications :
- `@OneToOne` indique une relation un-à-un entre `Course` et `CourseDetail`.
- `@JoinColumn(name = "course_id")` crée une colonne `course_id` dans la table `course_details` qui référence la clé primaire de `courses`.
- `@MapsId` signifie : « la clé primaire de CourseDetail est aussi la clé étrangère vers Course ». Cela garantit que chaque `CourseDetail` est lié à exactement un `Course`, et que les deux partagent le même identifiant.

### 8.4 Récursion JSON et DTOs

Quand vous exposez `Course` avec `lessons`, `students`, `courseDetail`, vous vous exposez au piège de la récursion : `Course` → `Lesson` → `Course` → `Lesson`… Si cela arrive, Spring affiche une erreur de type `StackOverflowError`. Résolutions possibles :
1. `@JsonManagedReference/@JsonBackReference` comme ci-dessus.
2. `@JsonIgnore` sur un des côtés.
3. DTO dédiés (`CourseDto`, `LessonDto`) pour contrôler exactement ce que vous exposez.

---

## Partie 9 — Exposer les relations via l’API

Vous savez maintenant comment gérer les relations entre entités. Il ne reste qu'à compléter l'application pour exposer ces relations via l'API. Créez les endpoints suivants :
- `GET /courses/{id}/lessons` : liste les leçons d’un cours
- `POST /courses/{id}/lessons` : ajoute une leçon à un cours
- `DELETE /courses/{courseId}/lessons/{lessonId}` : supprime une leçon d’un cours
- `GET /students/` : liste les étudiants
- `POST /students/` : crée un étudiant
- `GET /students/{id}/courses` : liste les cours suivis par un étudiant
- `GET /courses/{courseId}/students` : liste les étudiants inscrits à un cours
- `POST /courses/{courseId}/students/{studentId}` : inscrit un étudiant à un cours
- `DELETE /courses/{courseId}/students/{studentId}` : désinscrit un étudiant d’un cours
- `POST /courses/{id}/detail` : crée ou met à jour le détail d’un cours
- `GET /courses/{id}/detail` : affiche le détail d’un cours
