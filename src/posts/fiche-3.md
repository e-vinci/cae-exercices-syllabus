---
title: Fiche 3
description: Spring et sécurité.
permalink: posts/{{ title | slug }}/index.html
date: '2026-02-13'
tags: [fiche, spring, security]
---

# Spring Security & JWT

## Pourquoi cette fiche ?

Dans la fiche précédente nous avons branché notre API Spring Boot sur PostgreSQL via JPA : entités, repositories, relations et validations. Nous allons maintenant sécuriser cette application pour offrir une authentification robuste et des autorisations fines. L’objectif est double :

1. Séparer clairement authentification (qui est l’utilisateur ?) et autorisation (que peut‑il faire ?).
2. Appliquer Spring Security, JWT et une gestion saine des secrets sur une application existante sans casser ses fonctionnalités métiers.

### Objectifs pédagogiques

- Installer et configurer Spring Security, JWT (auth0 java-jwt) et spring-dotenv.
- Hacher les mots de passe.
- Implémenter un login qui renvoie un JWT, le vérifie dans un filtre, et bloque les requêtes illicites.
- Sécuriser les endpoints avec des rôles et des permissions.
- Externaliser les secrets.

---

## Partie 1 — Contexte & rappels

### 1.1 Authentification vs Autorisation

- **Authentification** : vérifier l’identité.
- **Autorisation** : vérifier les droits. On n’autorise jamais sans authentifier d’abord.

Nous parlerons de Role Based Access Control (RBAC). Les utilisateurs ont des rôles (ex: `USER`, `ADMIN`) qui déterminent ce qu’ils peuvent faire. Un `USER` peut consulter les pizzas, mais seul un `ADMIN` peut en créer ou supprimer. 

### 1.2 Architecture actuelle

Nous repartons de l’API de cours universitaire dévelopée dans la fiche précédente. Les endpoints existent déjà, mais sans sécurité. Notre travail consiste à ajouter les couches sécurité/transversal sans toucher inutilement au métier.

---

## Partie 2 — Repartir de la fin de la fiche 2

Plutôt que de cloner un nouveau dépôt, nous allons capitaliser sur le code produit à la fiche précédente :

1. Créez un nouveau module Spring Boot (par exemple `fiche3`). **Mettez la version de Java à 25** (nous allons utiliser certaines fonctionnalités récentes). Gardez les mêmes dépendances qu’en fiche 2 (Spring Web, Spring Data JPA, PostgreSQL, Validation), et ajoutez les nouvelles dépendances :
    - Spring Security (`spring-boot-starter-security`)
    - JWT (`com.auth0:java-jwt`)
    - Dotenv (`me.paulschwarz:spring-dotenv`)
2. Copiez-collez les packages `controllers`, `services`, `repositories`, `models` et `resources` du projet fiche 2 vers ce nouveau module. Si vous n'êtes pas arrivé au bout ou n'êtes pas certain de votre résultat, récupérez la base de départ sur [GitHub](https://github.com/e-vinci/cae-exercices-examples/tree/main/fiche2) puis reproduisez la copie.
3. Vérifiez que l’application démarre toujours et que les tests passent toujours.

---

## Partie 3 — Mettre en place l’authentification sans sécurité

### Créer l'entité `User`

La version fiche 2 n’avait pas encore de table `users`. Pour pouvoir authentifier quelqu’un, il nous faut :

- Une entité JPA persistée dans Postgres.
- Un repository pour la rechercher par `username`.
- Un service et un contrôleur pour gérer l'authentification.

Commencez par créer `User` dans `models` :

```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true)
    private String username;

    @Column(nullable = false)
    private String password; // hashé

    @Column(nullable = false)
    private String role; // USER ou ADMIN

    public User() {}

    // getters/setters
}
```

- `username` doit être unique pour identifier l’utilisateur lors du login.
- `password` ne peut **jamais** contenir le mot de passe en clair (voir Partie 6).
- `role` stocke la couche d’autorisation minimale (`USER`, `ADMIN`). On peut commencer simple avant d’introduire des rôles multiples.

### Créer le repository

Créez ensuite `UserRepository` avec une méthode de recherche par `username` :

```java
@Repository
public interface UserRepository extends CrudRepository<User, Long> {
    User findByUsername(String username);
}
```

### Créer des DTO

Le projet contient deux packages : `models` (entités JPA) et `dtos`. On ne veut pas exposer directement les entités JPA (id, password hash, etc.) aux contrôleurs. Les DTO servent d’objets simples (POJO) dédiés aux échanges HTTP.

Dans ce contexte :

- Les **entités** représentent l’état persistant exact (avec `id`, `role`, `password` hashé).
- Les **DTO** limitent les champs que le client peut envoyer ou recevoir (ex: un `Credentials` DTO contient seulement `username` + `password` en clair pour la phase de login).

Créez deux DTO :
- `Credentials` (username + password en clair) pour le login.
- `AuthenticatedUser` pour représenter la réponse du login (username + token).

Pour se faciliter le travail, nous allons utiliser des record - un type introduit récemment en java:

```java
public record Credentials(
        @NotBlank String username,
        @NotBlank String password
) {}
```

```java
public record AuthenticatedUser(
        @NotBlank String username,
        @NotBlank String token
) {}
```

Attention pour rappel:

- Les records sont immutables - ils n'ont donc pas de setter (on créer l'objet complet dans le constructeur)
- Les "getters" sont simplement le nom de la propriété avec des parenthèses (ex: username(), pas getUsername())

### Créer le service

Créez un `UserService` pour gérer l’inscription et le login (sans sécurité pour l’instant, on ajoutera ça plus tard).

```java
@Service
public class UserService {
    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public void register(Credentials credentials) {
        User user = new User();
        user.setUsername(credentials.username());
        user.setPassword(credentials.password()); // à remplacer par le hash plus tard
        user.setRole("USER");
        userRepository.save(user);
    }

    public AuthenticatedUser login(Credentials credentials) {
        User user = userRepository.findByUsername(credentials.username());
        if (user == null) {
            return null; // utilisateur inconnu
        }
        if (!user.getPassword().equals(credentials.password())) { // à remplacer par la vérification du hash plus tard
            return null; // mot de passe incorrect
        }

        AuthenticatedUser authUser = new AuthenticatedUser(user.getUsername(), "fake-jwt-token");
        return authUser;
    }
}
```

### Créer le contrôleur

Créez enfin un `UserController` pour exposer les endpoints d’inscription et de login :

```java
@RestController
@RequestMapping("/auths")
public class UserController {

    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @PostMapping("/register")
    @ResponseStatus(HttpStatus.CREATED)
    public void register(@Valid @RequestBody Credentials credentials) {
        userService.register(credentials);
    }

    @PostMapping("/login")
    public AuthenticatedUser login(@Valid @RequestBody Credentials credentials) {
        AuthenticatedUser authUser = userService.login(credentials);
        if (authUser == null) {
            throw new ResponseStatusException(HttpStatus.UNAUTHORIZED);
        }
        return authUser;
    }
}
```

---

## Partie 4 — Hashage des mots de passe

On ne stocke jamais un mot de passe en clair. Ajoutez un encodeur `BCryptPasswordEncoder` dans votre service utilisateur pour hacher les mots de passe à l’inscription et les vérifier à la connexion :

```java
@Service
public class UserService {
    private final UserRepository userRepository;
    private final BCryptPasswordEncoder passwordEncoder;

    public UserService(UserRepository userRepository, BCryptPasswordEncoder passwordEncoder) {
        this.userRepository = userRepository;
        this.passwordEncoder = passwordEncoder;
    }

    public void register(Credentials credentials) {
        User user = new User();
        user.setUsername(credentials.username());
        user.setPassword(passwordEncoder.encode(credentials.password()));
        user.setRole("USER");
        userRepository.save(user);
    }

    public AuthenticatedUser login(Credentials credentials) {
        User user = userRepository.findByUsername(credentials.username());
        if (user == null) {
            return null; // utilisateur inconnu
        }
        if (!passwordEncoder.matches(credentials.getPassword(), user.getPassword())) {
            return null; // mot de passe incorrect
        }

        AuthenticatedUser authUser = new AuthenticatedUser(user.getUsername(), "fake-jwt-token"); // à remplacer par le vrai token plus tard
        return authUser;
    }
}
```

Le `BcryptPasswordEncoder` nécessite une configuration pour être injecté dans le service. Par défaut, Spring ne sait pas comment en créer un. Il faut créer un *bean*, une méthode annotée `@Bean` qui retourne une instance de `BCryptPasswordEncoder` et explique à Spring comment en construire un. Créer une classe de configuration dédiée dans un package `config` :

```java
@Configuration
public class BcryptConfiguration {
    @Bean
    public BCryptPasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

Ajoutez un `CommandLineRunner` pour créer `admin/admin` et `user/user`, puis ouvrez la table `users` pour vérifier que les hashes commencent bien par `$2a$`.

```java
@Bean
public CommandLineRunner initUsers(UserRepository userRepository, BCryptPasswordEncoder passwordEncoder) {
    return args -> {
        User admin = new User();
        admin.setUsername("admin");
        admin.setPassword(passwordEncoder.encode("admin"));
        admin.setRole("ADMIN");
        userRepository.save(admin);

        User user = new User();
        user.setUsername("user");
        user.setPassword(passwordEncoder.encode("user"));
        user.setRole("USER");
        userRepository.save(user);
    };
}
```

---

## Partie 5 — Générer & vérifier un JWT

Ajoutez les constantes au début de `UserService` pour la configuration du JWT :

```java
private static final String JWT_SECRET = "myverysecretkey"; // à externaliser plus tard
private static final long JWT_LIFETIME = 24 * 60 * 60 * 1000; // en ms, 24h
private static final Algorithm ALGORITHM = Algorithm.HMAC256(JWT_SECRET);
```

```java
public AuthenticatedUser login(Credentials credentials) {
    User user = userRepository.findByUsername(credentials.username());
    if (user == null || !passwordEncoder.matches(credentials.password(), user.getPassword())) {
        return null; // utilisateur inconnu ou mot de passe incorrect
    }
    return createJwtToken(credentials.username());
}

public AuthenticatedUser createJwtToken(String username) {
    String token = JWT.create()
        .withIssuer("auth0")
        .withClaim("username", username)
        .withIssuedAt(new Date())
        .withExpiresAt(new Date(System.currentTimeMillis() + JWT_LIFETIME))
        .sign(ALGORITHM);

    AuthenticatedUser result = new AuthenticatedUser(username, token);
    return result;
}

public String verifyJwtToken(String token) {
    try {
        return JWT.require(ALGORITHM).build().verify(token).getClaim("username").asString();
    } catch (Exception e) {
        return null;
    }
}
```


---

## Partie 6 — Filtre JWT et gestion des erreurs

Nous avons désormais un endpoint de login qui génère un JWT valide. Il nous faut à présent un mécanisme pour vérifier ce token à chaque requête entrante.

Spring Security fonctionne avec une chaîne de filtres (FilterChain). Nous allons créer un filtre personnalisé qui s’exécutera avant les contrôles d’accès, pour extraire le JWT du header `Authorization`, le vérifier, et injecter l’identité de l’utilisateur dans le contexte de sécurité. 

Créez `JwtAuthenticationFilter` dans le package `config` :

```java
@Configuration
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    private final UserService userService;

    public JwtAuthenticationFilter(UserService userService) {
        this.userService = userService;
    }

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain) throws ServletException, IOException {
        String token = request.getHeader("Authorization");
        if (token == null) {
            filterChain.doFilter(request, response); // pas de token, on laisse Spring Security décider
            return;
        }

        String username = userService.verifyJwtToken(token);
        if (username == null) { // token invalide ou expiré
            response.sendError(HttpServletResponse.SC_UNAUTHORIZED, "Invalid or expired JWT");
            return; // renvoie un 401 et arrête la chaîne de filtres
        }

        User user = userService.readOneFromUsername(username);
        if (user == null) { // l'utilisateur n'existe plus
            response.sendError(HttpServletResponse.SC_UNAUTHORIZED, "User not found");
            return; // renvoie un 401 et arrête la chaîne de filtres
        }

        // token valide, la requête peut continuer
        // l'objet `authentication` contient l'identité de l'utilisateur et ses rôles, et est stocké dans le `SecurityContext` pour être accessible partout dans l'application

        List<GrantedAuthority> authorities = new ArrayList<>();
        authorities.add(new SimpleGrantedAuthority("ROLE_USER")); // tous les utilisateurs ont au moins ce rôle
        if ("ADMIN".equals(user.getRole())) {
            authorities.add(new SimpleGrantedAuthority("ROLE_ADMIN")); // les admins ont aussi ce rôle
        }
        
        UsernamePasswordAuthenticationToken authentication =
            new UsernamePasswordAuthenticationToken(user, null, authorities);
        authentication.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
        SecurityContextHolder.getContext().setAuthentication(authentication);

        filterChain.doFilter(request, response); // continue la chaîne de filtres
    }
}
```

Ce filtre vérifie la présence du token, le valide, et injecte l’utilisateur dans le contexte de sécurité. En cas d’échec (token manquant, invalide ou utilisateur introuvable), il renvoie un 401 et bloque la requête.

Notez que la méthode `readOneFromUsername` n'existe pas encore - il faut la rajouter sur le `UserService`.

---

## Partie 7 — Configurer Spring Security

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity()
public class SecurityConfiguration {

    private final JwtAuthenticationFilter jwtAuthenticationFilter;

    public SecurityConfiguration(JwtAuthenticationFilter jwtAuthenticationFilter) {
        this.jwtAuthenticationFilter = jwtAuthenticationFilter;
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            .csrf(AbstractHttpConfigurer::disable)
            .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class)
            .build();
    }
}
```

Explications :
- `csrf` est désactivé car nous n’utilisons pas de sessions ni de formulaires, donc pas de risque de problèmes de sécurité CSRF.
- `sessionManagement` est configuré en `STATELESS` car nous utilisons des JWT, pas de sessions serveur.
- `addFilterBefore` place notre `JwtAuthenticationFilter` avant le filtre de Spring Security qui gère l’authentification par formulaire (UsernamePasswordAuthenticationFilter). Ainsi, notre filtre s’exécutera à chaque requête pour vérifier le JWT.
- `@EnableMethodSecurity` permet d’utiliser des annotations de sécurité au niveau des méthodes. Cela nous permettra de sécuriser certains endpoints avec des rôles spécifiques.
- `@EnableWebSecurity` active la configuration de sécurité web, cela ne fonctionne pas sans.

### Test

Un fichier .http va nous permettre de tester ce endpoint:

```http

### Login with valid credentials (admin/admin)
POST http://localhost:8080/auths/login
Content-Type: application/json

{
  "username": "admin",
  "password": "admin"
}

### Login with incorrect password (should return 401 Unauthorized)
POST http://localhost:8080/auths/login
Content-Type: application/json

{
  "username": "admin",
  "password": "wrongpassword"
}
```

---

## Partie 8 — Sécuriser les endpoints selon les rôles

Maintenant que nous avons un mécanisme d’authentification en place, nous pouvons sécuriser les endpoints selon les rôles. Nous allons utiliser l’annotation `@PreAuthorize` pour spécifier les règles d’accès à chaque endpoint. Nous allons appliquer les règles suivantes :
- La consultation des cours est ouverte à tous les utilisateurs.
- La consultation des détails du cours, des leçons et des étudiants d’un cours est réservée aux utilisateurs authentifiés.
- La création, la modification et la suppression des cours, de leurs détails, et des leçons sont réservées aux administrateurs.
- L’inscription et la désinscription à un cours sont possibles pour les utilisateurs eux-mêmes ou les administrateurs.

Dans le `CourseController`, ajoutez les annotations de sécurité :

```java
@RestController
@RequestMapping("/courses")
public class CourseController {

    // n'importe quel utilisateur authentifié ou non peut consulter la liste des cours
    @GetMapping("/")
    public Iterable<Course> listCourses() {
        return coursesService.getAllCourses();
    }

    // n'importe quel utilisateur authentifié ou non peut consulter un cours
    @GetMapping("/{id}")
    public Course getCourse(@PathVariable long id) {
        return coursesService.getCourse(id)
            .orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND, "Course not found"));
    }

    @PostMapping("/")
    @ResponseStatus(HttpStatus.CREATED)
    @PreAuthorize("hasRole('ADMIN')") // seul un ADMIN peut créer un cours
    public Course createCourse(@Valid @RequestBody Course course) {
        try {
            return coursesService.createCourse(course);
        } catch (IllegalStateException ex) {
            throw new ResponseStatusException(HttpStatus.CONFLICT, ex.getMessage());
        }
    }

    @PutMapping("/{id}")
    @PreAuthorize("hasRole('ADMIN')") // seul un ADMIN peut modifier un cours
    public Course updateCourse(@PathVariable long id, @Valid @RequestBody CourseUpdateDTO payload) {
        try {
            return coursesService.updateCourse(id, payload);
        } catch (IllegalStateException ex) {
            throw new ResponseStatusException(HttpStatus.CONFLICT, ex.getMessage());
        } catch (NoSuchElementException ex) {
            throw new ResponseStatusException(HttpStatus.NOT_FOUND, ex.getMessage());
        }
    }

    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    @PreAuthorize("hasRole('ADMIN')") // seul un ADMIN peut supprimer un cours
    public void deleteCourse(@PathVariable long id) {
        try {
            coursesService.deleteCourse(id);
        } catch (NoSuchElementException ex) {
            throw new ResponseStatusException(HttpStatus.NOT_FOUND, ex.getMessage());
        }
    }

    @GetMapping("/{id}/lessons")
    @PreAuthorize("isAuthenticated()") // seuls les utilisateurs authentifiés peuvent consulter les leçons d'un cours
    public List<Lesson> listLessons(@PathVariable long id) {
        return coursesService.getLessons(id);
    }

    @PostMapping("/{id}/lessons")
    @ResponseStatus(HttpStatus.CREATED)
    @PreAuthorize("hasRole('ADMIN')") // seul un ADMIN peut ajouter une leçon à un cours
    public Lesson addLesson(@PathVariable long id, @Valid @RequestBody LessonRequest payload) {
        try {
            return coursesService.addLesson(id, payload);
        } catch (NoSuchElementException ex) {
            throw new ResponseStatusException(HttpStatus.NOT_FOUND, ex.getMessage());
        }
    }

    @DeleteMapping("/{courseId}/lessons/{lessonId}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    @PreAuthorize("hasRole('ADMIN')") // seul un ADMIN peut supprimer une leçon d'un cours
    public void deleteLesson(@PathVariable long courseId, @PathVariable long lessonId) {
        try {
            coursesService.removeLesson(courseId, lessonId);
        } catch (NoSuchElementException ex) {
            throw new ResponseStatusException(HttpStatus.NOT_FOUND, ex.getMessage());
        }
    }

    @GetMapping("/{id}/students")
    @PreAuthorize("isAuthenticated()") // seuls les utilisateurs authentifiés peuvent consulter les étudiants d'un cours
    public Set<Student> listStudents(@PathVariable long id) {
        return coursesService.getStudents(id);
    }

    @PostMapping("/{courseId}/students/{studentId}")
    @ResponseStatus(HttpStatus.CREATED)
    @PreAuthorize("isAuthenticated()") // seul un ADMIN ou l'utilisateur lui-même peut s'inscrire à un cours
    public Course enrollStudent(@PathVariable long courseId, @PathVariable long studentId, @AuthenticationPrincipal User currentUser) {
        if (!Objects.equals(currentUser.getId(), studentId) && !Objects.equals(currentUser.getRole(), "ADMIN")) {
            throw new ResponseStatusException(HttpStatus.FORBIDDEN, "You can only enroll yourself or you must be an admin");
        }
        try {
            return coursesService.enrollStudent(courseId, studentId);
        } catch (NoSuchElementException ex) {
            throw new ResponseStatusException(HttpStatus.NOT_FOUND, ex.getMessage());
        }
    }

    @DeleteMapping("/{courseId}/students/{studentId}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    @PreAuthorize("isAuthenticated()") // seul un ADMIN ou l'utilisateur lui-même peut se désinscrire d'un cours
    public void unenroll(@PathVariable long courseId, @PathVariable long studentId, @AuthenticationPrincipal User currentUser) {
        if (!Objects.equals(currentUser.getId(), studentId) && !Objects.equals(currentUser.getRole(), "ADMIN")) {
            throw new ResponseStatusException(HttpStatus.FORBIDDEN, "You can only unenroll yourself or you must be an admin");
        }
        try {
            coursesService.unenrollStudent(courseId, studentId);
        } catch (NoSuchElementException ex) {
            throw new ResponseStatusException(HttpStatus.NOT_FOUND, ex.getMessage());
        }
    }

    @GetMapping("/{id}/detail")
    @PreAuthorize("isAuthenticated()") // seuls les utilisateurs authentifiés peuvent consulter les détails d'un cours
    public CourseDetail getDetail(@PathVariable long id) {
        return coursesService.getDetail(id)
            .orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND, "Course detail not found"));
    }

    @PostMapping("/{id}/detail")
    @ResponseStatus(HttpStatus.CREATED)
    @PreAuthorize("hasRole('ADMIN')") // seul un ADMIN peut créer ou mettre à jour les détails d'un cours
    public CourseDetail createOrUpdateDetail(@PathVariable long id, @Valid @RequestBody CourseDetailRequest payload) {
        try {
            return coursesService.upsertDetail(id, payload);
        } catch (NoSuchElementException ex) {
            throw new ResponseStatusException(HttpStatus.NOT_FOUND, ex.getMessage());
        }
    }
}
```

Explications :
- `@PreAuthorize` permet de spécifier des expressions SpEL (Spring Expression Language) pour contrôler l’accès à chaque endpoint.
- `hasRole('ADMIN')` signifie que seul un utilisateur avec le rôle `ADMIN` peut accéder à cet endpoint.
- `isAuthenticated()` signifie que n’importe quel utilisateur authentifié (qu’il soit `USER` ou `ADMIN`) peut accéder à cet endpoint.
- Pour les endpoints d’inscription/désinscription, nous vérifions que l’utilisateur est soit l’utilisateur lui-même, soit un administrateur. Nous utilisons `@AuthenticationPrincipal` pour injecter l’utilisateur récupéré depuis son token par le filtre dans la méthode. Cela nous permet de faire des contrôles d’accès basés sur l’identité de l’utilisateur. Si l’utilisateur essaie de s’inscrire ou de se désinscrire pour un autre utilisateur sans être admin, nous renvoyons un 403 Forbidden.

Ajoutez des règles similaires dans les autres contrôleurs pour sécuriser les endpoints selon les rôles.

---

## Partie 9 — Protéger les secrets avec `.env`

Certaines informations sont des secrets : des informations sensibles qui doivent rester confidentielles. C'est le cas ici des informations de connexion de la base de données et du secret JWT. Il est crucial de ne jamais les hardcoder dans le code source, car cela peut entraîner des fuites de sécurité si le code est partagé ou publié. Une erreur commune est de commit ces informations. Une fois que c'est le cas, il est trop tard. Les hackers peuvent alors accéder à ces informations sensibles, même si elles sont supprimées du dépôt par la suite. Nous allons voir deux méthodes pour externaliser ces secrets : les variables d’environnement et les fichiers `.env`.

### 9.1 Variables d’environnement

Les variables d’environnement sont une méthode simple et efficace pour stocker des secrets. Elles sont gérées par le système d’exploitation et ne font pas partie du code source. Il faut commencer par supprimer le secret JWT de `UserService` et la configuration de la base de données dans `application.properties`, puis les remplacer par des références à des variables d’environnement.

Modifiez dans `application.properties` :

```properties
spring.datasource.password=${DB_PASSWORD}
```

Modifiez dans `UserService` :

```java
private static final String JWT_SECRET;
static {
    JWT_SECRET = System.getenv("JWT_SECRET");
    if (JWT_SECRET == null || JWT_SECRET.isBlank()) {
        throw new IllegalStateException("JWT_SECRET environment variable not set");
    }
}
```

Lancez ensuite l’application en définissant les variables d’environnement nécessaires :

- Sous macOS/Linux :

```bash
export DB_PASSWORD=cae
export JWT_SECRET=verysecretkeyandverylongsoastobeabletosustainabrute-force-attack
./mvnw spring-boot:run 
```

- Sous Windows PowerShell :

```powershell
$Env:DB_PASSWORD = "cae"
$Env:JWT_SECRET = "verysecretkeyandverylongsoastobeabletosustainabrute-force-attack"
./mvnw spring-boot:run
```

### 9.2 Fichier `.env`

Plutôt que de définir manuellement les variables d’environnement à chaque lancement, nous pouvons utiliser un fichier `.env` pour stocker ces secrets de manière structurée. Nous allons utiliser la bibliothèque `spring-dotenv` pour charger automatiquement les variables d’environnement depuis ce fichier.

Créez un fichier `.env` à la racine du projet. Ajoutez-le directement au `.gitignore` pour éviter de le committer! Dans ce fichier, ajoutez les secrets :

```
DB_HOST=localhost
DB_PORT=5433
DB_NAME=cae_db
DB_USER=cae_user
DB_PASSWORD=cae
JWT_SECRET=verysecretkeyandverylongsoastobeabletosustainabrute-force-attack
```

Spring-dotenv lit automatiquement le fichier `.env` et alimente automatiquement l’environnement. 

Pour la configuration de la base de données, il suffit de mettre à jour `application.properties` pour référencer la variable d’environnement :

```properties
spring.datasource.url=jdbc:postgresql://${DB_HOST:localhost}:${DB_PORT:5432}/${DB_NAME}
spring.datasource.username=${DB_USER}
spring.datasource.password=${DB_PASSWORD}
```

Nous pouvons ensuite permettre au code d'accéder aux variables d’environnement via des annotations. Ajoutez la configuration pour charger ce fichier dans la classe principale de votre application Spring Boot :

```java
public class Fiche3Application {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
        DotenvPropertySource.addToEnvironment(context.getEnvironment());
        SpringApplication.run(Fiche3Application.class, args);
    }
}
```

Modifiez ensuite `UserService` pour utiliser `@Value` et injecter le secret depuis l’environnement. Cette variable ne peut plus être `static` car elle est injectée par Spring, et doit être une propriété d’instance :

```java
@Value("${JWT_SECRET}")
private String jwtSecret;
```

---

# Tests

Il est important de tester les fonctionnalités de sécurité pour s’assurer que les règles d’accès sont correctement appliquées. Voici quelques scénarios de test à considérer :

1. **Test d’inscription** : Vérifiez que les utilisateurs peuvent s’inscrire avec des identifiants valides et que les mots de passe sont correctement hashés.
2. **Test de login** : Vérifiez que les utilisateurs peuvent se connecter avec des identifiants valides et que le JWT généré est valide.
3. **Test d’accès aux endpoints** : Vérifiez que les utilisateurs avec le rôle `USER` peuvent accéder aux endpoints autorisés et que les utilisateurs sans authentification ou avec des rôles insuffisants sont correctement bloqués.
4. **Test de token expiré** : Vérifiez que les requêtes avec un token expiré sont rejetées avec un 401 Unauthorized.
5. **Test de token invalide** : Vérifiez que les requêtes avec un token malformé ou signé avec une clé incorrecte sont rejetées avec un 401 Unauthorized.
6. **Test de désinscription** : Vérifiez que les utilisateurs peuvent se désinscrire d’un cours et que les règles d’accès sont respectées.
7. **Test de rôles** : Vérifiez que les utilisateurs avec le rôle `ADMIN` peuvent accéder à tous les endpoints, tandis que les utilisateurs avec le rôle `USER` ont des accès limités.
8. **Test de secrets** : Vérifiez que les secrets ne sont pas exposés dans le code source et que l’application fonctionne correctement avec les secrets externalisés.
