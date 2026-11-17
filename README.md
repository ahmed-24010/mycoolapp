# TP01 — mycoolapp

Première application **Spring Boot** réalisée dans le cadre du TP01 (24010) : une petite API REST qui expose quelques endpoints, des propriétés personnalisées, la sécurité par défaut de Spring Security et les endpoints de supervision d'Actuator.

## Stack technique

| Élément | Version / détail |
|---|---|
| Java | 25 |
| Spring Boot | 4.0.8 |
| Build | Maven (wrapper `mvnw` fourni) |
| Dépendances | `spring-boot-starter-webmvc`, `-actuator`, `-security`, `-devtools`, `-webmvc-test` (tests) |

## Structure du dépôt

```text
TP1_24010/
├── README.md
└── mycoolapp/mycoolapp/
    ├── pom.xml
    ├── mvnw, mvnw.cmd                  # Maven wrapper
    ├── REPONSES.md                     # réponses aux questions de compréhension
    ├── Rapport_TP01_24010.pdf          # rapport du TP
    └── src/
        ├── main/java/com/tp1/mycoolapp/
        │   ├── MycoolappApplication.java   # point d'entrée (@SpringBootApplication)
        │   └── FunRestController.java      # endpoints REST
        ├── main/resources/application.properties
        └── test/java/com/tp1/mycoolapp/MycoolappApplicationTests.java
```

## Prérequis

- JDK 25
- Aucune installation de Maven nécessaire (utiliser `mvnw`)

## Lancer l'application

Depuis le dossier `mycoolapp/mycoolapp` :

```bash
# Mode développement (DevTools actif)
./mvnw spring-boot:run          # Windows : mvnw.cmd spring-boot:run

# Ou : construire puis lancer le fat-jar
./mvnw clean package
java -jar target/mycoolapp-0.0.1-SNAPSHOT.jar
```

L'application démarre sur <http://localhost:8080>.

Pour changer le port sans recompiler :

```bash
java -jar target/mycoolapp-0.0.1-SNAPSHOT.jar --server.port=9090
```

## Tests

```bash
./mvnw test
```

## Endpoints

Spring Security étant dans le classpath, **tous les endpoints exigent une authentification HTTP Basic**. Les identifiants sont définis dans `application.properties` (`spring.security.user.name` / `spring.security.user.password`).

### API applicative

| Méthode | URL | Réponse |
|---|---|---|
| GET | `/` | `Hello World!` |
| GET | `/teaminfo` | `Coach: <coach.name>, Team name: <team.name>` |
| GET | `/workout` | `Run a hard 5k!` |

Exemple :

```bash
curl -u <utilisateur>:<mot_de_passe> http://localhost:8080/teaminfo
```

### Actuator

Tous les endpoints Actuator sont exposés (`management.endpoints.web.exposure.include=*`), par exemple :

- `/actuator/health`
- `/actuator/info` (affiche les propriétés `info.app.*`)
- `/actuator/beans`, `/actuator/env`, …

## Configuration

Fichier : [application.properties](mycoolapp/mycoolapp/src/main/resources/application.properties)

| Propriété | Rôle |
|---|---|
| `server.port` | Port HTTP (8080) |
| `coach.name`, `team.name` | Propriétés personnalisées injectées avec `@Value` |
| `spring.security.user.*` | Identifiants de l'utilisateur HTTP Basic |
| `management.endpoints.web.exposure.include` | Endpoints Actuator exposés |
| `management.info.env.enabled`, `info.app.*` | Contenu de `/actuator/info` |

## Avertissement sécurité

Ce projet est un **TP pédagogique**. Le mot de passe est écrit en clair dans `application.properties` et tous les endpoints Actuator sont exposés. Ne pas reproduire cette configuration en production : utiliser des variables d'environnement pour les secrets et n'exposer que `health` et `info`.

## Documentation complémentaire

- [REPONSES.md](mycoolapp/mycoolapp/REPONSES.md) : réponses aux questions de compréhension (`@SpringBootApplication`, auto-configuration, `spring-boot:run` vs `java -jar`, priorité des propriétés, Actuator, `@RestController`).
- [Rapport_TP01_24010.pdf](mycoolapp/mycoolapp/Rapport_TP01_24010.pdf) : rapport du TP.
