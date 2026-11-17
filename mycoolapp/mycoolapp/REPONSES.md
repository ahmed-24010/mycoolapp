# TP01 — Réponses aux questions de compréhension

## 1. Les trois annotations de `@SpringBootApplication`

`@SpringBootApplication` regroupe :

- **`@Configuration`** : la classe devient une source de définitions de beans (méthodes `@Bean`). Elle sert de classe de configuration principale.
- **`@EnableAutoConfiguration`** : active l'auto-configuration. Spring Boot configure automatiquement des beans selon les dépendances présentes dans le classpath (Tomcat, Jackson, Spring MVC, etc.).
- **`@ComponentScan`** : scanne le package de la classe principale **et ses sous-packages** pour trouver les composants (`@RestController`, `@Service`, `@Component`, `@Repository`...).

**Contrôleur dans un package frère** : si la classe principale est dans `com.tp1.mycoolapp` et le contrôleur dans `com.tp1.autre`, le `@ComponentScan` ne le voit pas. Le contrôleur n'est pas enregistré comme bean, ses routes n'existent pas, et on obtient une erreur 404. Il faut soit le déplacer dans un sous-package, soit déclarer explicitement `scanBasePackages` sur `@SpringBootApplication`.

## 2. Auto-configuration

L'auto-configuration est le mécanisme par lequel Spring Boot crée et configure automatiquement les beans nécessaires, en fonction des classes présentes dans le classpath, des propriétés définies et des beans déjà déclarés. Elle évite la configuration XML ou Java manuelle.

**Exemple** : en ajoutant simplement `spring-boot-starter-webmvc`, on obtient sans écrire de configuration : Spring MVC avec la `DispatcherServlet` enregistrée, un serveur **Tomcat embarqué** démarré sur le port 8080, Jackson pour la conversion JSON, et la gestion des erreurs par défaut.

## 3. `./mvnw spring-boot:run` vs `java -jar`

- `./mvnw spring-boot:run` lance l'application via Maven, à partir des classes compilées et des dépendances du projet (pas de JAR final). C'est un mode développement.
- `java -jar target/mycoolapp-0.0.1-SNAPSHOT.jar` lance le **fat-jar** exécutable, qui contient l'application, ses dépendances et Tomcat. C'est le mode utilisé en déploiement, sans Maven.

**DevTools** est actif avec `spring-boot:run` (et depuis l'IDE). Il est **désactivé avec `java -jar`** : Spring Boot détecte qu'il s'agit d'une application packagée (classloader de production) et considère que c'est un environnement de production. Le redémarrage automatique et les réglages de développement n'ont pas de sens dans ce cas, et DevTools est d'ailleurs exclu du JAR par défaut.

## 4. Mot de passe généré par Spring Security

Sans configuration, Spring Boot crée un utilisateur `user` avec un mot de passe aléatoire (UUID) affiché dans la console (`Using generated security password: ...`), **régénéré à chaque démarrage**.

C'est un bon comportement par défaut car il n'existe aucun mot de passe connu ou codé en dur : un attaquant ne peut pas deviner des identifiants par défaut, et l'application n'est jamais livrée ouverte ni avec un mot de passe public (type `admin/admin`).

On le personnalise (`spring.security.user.name` / `spring.security.user.password`) parce qu'un mot de passe qui change à chaque redémarrage est inutilisable en pratique : les clients, scripts de supervision et tests ont besoin d'identifiants stables. (En production, on utilise de vrais utilisateurs en base ou un fournisseur d'identité, et le secret est fourni par variable d'environnement, jamais en clair dans le dépôt.)

## 5. Ordre de priorité des sources de propriétés

Du plus fort au plus faible (vue simplifiée) :

1. Arguments de ligne de commande (`--server.port=9090`)
2. Variables d'environnement
3. `application.properties`
4. Valeurs par défaut

Si `application.properties` définit `server.port=8080` et qu'on lance avec `--server.port=9090`, c'est le **port 9090** qui est utilisé : l'argument de ligne de commande l'emporte.

Intérêt pratique : un **même JAR** peut être réutilisé dans plusieurs environnements (dev, staging, prod) en changeant seulement les arguments ou variables d'environnement, sans recompiler.

## 6. `management.info.env.enabled=true`

Cette propriété active le contributeur d'info « env » qui lit les propriétés commençant par `info.*` (ex. `info.app.name`) et les expose dans `/actuator/info`.

Sans cette ligne, `/actuator/info` renvoie un JSON vide : `{}`, car les propriétés `info.*` ne sont pas publiées.

## 7. Danger d'exposer `/actuator/beans` et `/actuator/env` sans authentification

- `/actuator/beans` révèle toute l'architecture interne : liste des beans, classes, dépendances. Cela facilite la reconnaissance et la recherche de failles pour un attaquant.
- `/actuator/env` expose les propriétés de configuration et les variables d'environnement : URLs de bases de données, noms d'utilisateurs, éventuellement des secrets (même si certaines valeurs sont masquées, des fuites restent possibles).

Sans authentification, n'importe qui sur le réseau peut lire ces informations. Il faut donc les protéger (Spring Security), n'exposer que les endpoints nécessaires (`health`, `info`) et idéalement placer Actuator sur un port ou un réseau interne.

**Remarque sur Actuator et Spring Security** : dès que `spring-boot-starter-security` est dans le classpath, Spring Security protège **tous** les endpoints par défaut, y compris `/actuator/health` et `/actuator/info`. Ils exigent donc l'authentification HTTP Basic (`abdou` / `cHk3694`), ce qui est normal dans ce TP. Rendre `health` public nécessiterait une configuration de sécurité explicite.

## 8. `@RestController` vs `@Controller`

- `@Controller` est un contrôleur Spring MVC classique : les valeurs retournées par les méthodes sont interprétées comme des **noms de vues** (templates) à résoudre. Pour renvoyer directement des données, chaque méthode doit être annotée `@ResponseBody`.
- `@RestController` = `@Controller` + `@ResponseBody` appliqué à toute la classe : la valeur retournée est écrite **directement dans le corps de la réponse HTTP** (texte, ou JSON via Jackson), sans passer par une vue. C'est l'annotation adaptée aux API REST.
