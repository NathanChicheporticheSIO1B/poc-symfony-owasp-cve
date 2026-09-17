## Étape 3 : 

Voici pour chaque module, la phrase en commentaire dans le contrôleur qui explique la faille volontaire :

/**  
 * OWASP A01:2021 - Broken Access Control. *  
 * - /lab/a01/invoices/{id} : IDOR, aucune vérification que la facture  
 *   consultée appartient bien à l'utilisateur connecté.  
 * - /lab/a01/admin : vérifie seulement qu'un utilisateur est connecté,  
 *   jamais son rôle -> élévation de privilèges verticale.  
 */

/**  
 * OWASP A02:2021 - Cryptographic Failures.  
 *  
 * - Mots de passe utilisateurs hashés en MD5 sans sel (voir App\Entity\User).  
 * - Jetons "API" (JWT) signés avec un secret faible, codé en dur, et sans  
 *   expiration : facilement rejouable, cassable hors-ligne, ou forgeable.  
 * - Utilise firebase/php-jwt en version volontairement ancienne (voir composer.json)  
 *   qui contient de vraies CVE connues (à retrouver avec Trivy / composer audit).  
 */

/**  
 * OWASP A03:2021 - Injection.  
 *  
 * - /lab/a03/search : injection SQL, la requête est construite par  
 *   concaténation de chaînes au lieu de paramètres liés.  
 * - /lab/a03/comments : XSS stockée, le contenu est ré-affiché avec le  
 *   filtre Twig |raw (aucun échappement) et sans aucune validation.  
 * - /lab/a03/ping : injection de commande, l'entrée utilisateur est passée  
 *   telle quelle à shell_exec().  
 */

/**  
 * OWASP A04:2021 - Insecure Design.  
 *  
 * /lab/a04/checkout fait confiance à un champ de formulaire caché envoyé  
 * par le client pour connaître le prix à débiter : aucun recalcul côté  
 * serveur à partir du catalogue -> manipulation de prix triviale.  
 *  
 * Voir aussi /login (Cours OWASP A04) : aucune limitation du nombre de  
 * tentatives de connexion, aucun CAPTCHA, aucun verrouillage de compte.  
 */

/**  
 * OWASP A05:2021 - Security Misconfiguration.  
 *  
 * - /lab/a05/phpinfo expose phpinfo() publiquement (versions, chemins, config serveur).  
 * - /_profiler et la barre de débogage Symfony restent actifs (voir .env : APP_ENV=dev  
 *   et APP_DEBUG=1 même dans l'image "de production").  
 * - /lab/a05/api/users renvoie du JSON avec un header CORS `Access-Control-Allow-Origin: *`.  
 * - Le Dockerfile ne définit aucun utilisateur non-root (voir docker/Dockerfile).  
 */

/**  
 * OWASP A06:2021 - Vulnerable and Outdated Components. *  
 * Cette page ne contient pas de faille exploitable elle-même : c'est  
 * l'ensemble du projet (composer.lock, Dockerfile) qui illustre cette  
 * catégorie. Voir composer.json pour les versions volontairement anciennes,  
 * et le Dockerfile pour l'image PHP de base obsolète.  
 *  
 * C'est la seule catégorie de ce TP que Trivy peut détecter automatiquement  
 * (analyse SCA) : les autres routes /lab/aXX relèvent du SAST/DAST/relecture  
 * manuelle de code, hors du périmètre de Trivy.  
 */

/**  
 * OWASP A07:2021 - Identification and Authentication Failures. *  
 * Les failles concrètes sont dans App\Controller\AuthController (/login,  
 * /register, /forgot-password) : pas de régénération de session, pas de  
 * limitation de tentatives, énumération d'utilisateurs, cookie "remember me"  
 * prévisible. Cette page se contente d'afficher l'état de session courant  
 * pour observer ces défauts en direct.  
 */

/**  
 * OWASP A08:2021 - Software and Data Integrity Failures. *  
 * - /lab/a08/preferences sérialise les préférences avec PHP serialize()  
 *   et les stocke dans un cookie en clair, puis les relit avec unserialize()  
 *   sans aucune validation -> désérialisation non sûre / PHP Object Injection.  
 * - /lab/a08/upload accepte n'importe quel fichier, sans vérifier son  
 *   extension ni son type MIME, et le stocke dans public/uploads/ (donc  
 *   directement accessible et potentiellement exécutable).  
 */

/**  
 * OWASP A09:2021 - Security Logging and Monitoring Failures. *  
 * - Les tentatives de connexion échouées journalisent le mot de passe fourni  
 *   en clair (voir App\Controller\AuthController::login) : une donnée  
 *   sensible se retrouve dans un fichier de log.  
 * - Cette page lit et affiche directement le fichier de log applicatif,  
 *   sans aucune authentification -> exposition de données sensibles à  
 *   quiconque connaît l'URL.  
 */

/**  
 * OWASP A10:2021 - Server-Side Request Forgery (SSRF). *  
 * /lab/a10/preview simule une fonctionnalité "aperçu de lien" : le serveur  
 * va chercher lui-même l'URL fournie par l'utilisateur et en affiche le  
 * contenu, sans aucune liste blanche ni filtrage des IP privées/internes.  
 * Depuis Docker, cette route peut par exemple interroger d'autres  
 * conteneurs du réseau interne (http://db:3306, etc.).  
 */

## Étape 4 : 

Voici le tableau de mon rapport :

| Library                 | Vulnerability  | Severity | Status  | Installed Version                                                                                                                       | Fixed Version                                                                                                                                                                                                          | Title                                                                                                                            |
| ----------------------- | -------------- | -------- | ------- | --------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| firebase/php-jwt        | CVE-2021-46743 | CRITICAL | fixed   | v5.5.1                                                                                                                                  | 6.0.0                                                                                                                                                                                                                  | Key/algorithm type confusion  <br>  <br>[https://avd.aquasec.com/nvd/cve-2021-46743](https://avd.aquasec.com/nvd/cve-2021-46743) |
| guzzlehttp/guzzle       | CVE-2026-69246 | HIGH     | 6.5.8   | 7.15.2, 8.0.1                                                                                                                           | Guzzle is an extensible PHP HTTP client. Prior to 7.15.2 and 8.0.1,...  <br>  <br>[https://avd.aquasec.com/nvd/cve-2026-69246](https://avd.aquasec.com/nvd/cve-2026-69246)                                             |                                                                                                                                  |
| symfony/cache           | CVE-2026-45073 | MEDIUM   | v6.1.11 | 6.3.0, 6.4.0, 7.1.07.4.0, 7.4.12, 5.2.0, 5.3.0, 6.1.0, 7.2.0, 7.3.0, 8.0.12, 3.0.0, 5.0.0, 5.4.0, 6.4.40, 6.2.0, 4.0.0, 5.1.0, 5.4.52   | Symfony is a PHP framework for web and console applications and a...  <br>  <br>[https://avd.aquasec.com/nvd/cve-2026-45073](https://avd.aquasec.com/nvd/cve-2026-45073)                                               |                                                                                                                                  |
| symfony/http-foundation | CVE-2025-64500 | HIGH     | v6.1.12 | 7.1.0, 7.3.0, 3.0.0, 5.2.0, 6.2.0, 7.3.7, 4.0.0, 6.4.0, 5.1.0, 5.3.0, 5.4.0, 6.1.0, 6.4.29, 7.2.0, 5.0.0, 5.4.50, 6.3.0                 | [Symfony is a PHP framework for web and console applications and a...  <br>  <br>](https://avd.aquasec.com/nvd/cve-2025-64500)[https://avd.aquasec.com/nvd/cve-2025-64500](https://avd.aquasec.com/nvd/cve-2025-64500) |                                                                                                                                  |
| symfony/monolog-bridge  | CVE-2026-45077 | CRITICAL | v6.1.11 | 5.1.0, 5.3.0, 7.2.0, 8.0.12, 5.2.0, 7.4.12, 3.0.0, 4.0.0, 5.4.0, 5.4.52, 6.1.0, 6.2.0, 5.0.0, 6.4.40, 7.3.0, 7.4.0, 6.3.0, 6.4.0, 7.1.0 | [Symfony is a PHP framework for web and console applications and a...  <br>  <br>](https://avd.aquasec.com/nvd/cve-2026-45077)[https://avd.aquasec.com/nvd/cve-2026-45077](https://avd.aquasec.com/nvd/cve-2026-45077) |                                                                                                                                  |
| symfony/security-http   | CVE-2026-45063 | CRITICAL | v6.1.12 | 5.4.0, 5.4.52, 7.1.0, 7.3.0, 7.4.0, 3.0.0, 6.3.0, 8.0.12, 5.3.0, 6.2.0, 7.4.12, 4.0.0, 6.1.0, 6.4.0, 6.4.40, 7.2.0, 5.0.0, 5.1.0, 5.2.0 | [Symfony is a PHP framework for web and console applications and a...  <br>  <br>](https://avd.aquasec.com/nvd/cve-2026-45063)[https://avd.aquasec.com/nvd/cve-2026-45063](https://avd.aquasec.com/nvd/cve-2026-45063) |                                                                                                                                  |
| symfony/security-http   | CVE-2026-48489 | HIGH     | v6.1.12 | 5.1.0, 5.2.0, 6.1.0, 6.3.0, 7.4.0, 4.0.0, 5.0.0, 6.2.0, 7.2.0, 3.0.0, 5.4.53, 6.4.0, 7.4.13, 8.0.13, 5.3.0, 5.4.0, 6.4.41, 7.1.0, 7.3.0 | [Symfony is a PHP framework for web and console applications and a...  <br>  <br>](https://avd.aquasec.com/nvd/cve-2026-48489)[https://avd.aquasec.com/nvd/cve-2026-48489](https://avd.aquasec.com/nvd/cve-2026-48489) |                                                                                                                                  |
| symfony/yaml            | CVE-2026-45133 | HIGH     | v6.1.11 | 7.4.0, 7.4.12, 5.3.0, 6.2.0, 7.1.0, 4.0.0, 5.0.0, 5.1.0, 5.4.0, 6.1.0, 6.3.0, 7.3.0, 6.4.0, 8.0.12, 3.0.0, 5.2.0, 5.4.52, 6.4.40, 7.2.0 | [Symfony is a PHP framework for web and console applications and a...  <br>  <br>](https://avd.aquasec.com/nvd/cve-2026-45133)[https://avd.aquasec.com/nvd/cve-2026-45133](https://avd.aquasec.com/nvd/cve-2026-45133) |                                                                                                                                  |
| symfony/yaml            | CVE-2026-45304 | HIGH     | v6.1.11 | 5.3.0, 6.1.0, 6.4.0, 8.0.12, 6.2.0, 6.3.0, 7.4.12, 4.0.0, 5.0.0, 5.1.0, 5.2.0, 7.3.0, 7.4.0, 3.0.0, 5.4.0, 5.4.52, 6.4.40, 7.1.0, 7.2.0 | [Symfony is a PHP framework for web and console applications and a...  <br>  <br>](https://avd.aquasec.com/nvd/cve-2026-45304)[https://avd.aquasec.com/nvd/cve-2026-45304](https://avd.aquasec.com/nvd/cve-2026-45304) |                                                                                                                                  |
| symfony/yaml            | CVE-2026-45305 | HIGH     | v6.1.11 | 6.4.0, 5.1.0, 7.4.12, 8.0.12, 3.0.0, 5.2.0, 5.4.0, 7.4.0, 4.0.0, 5.0.0, 5.3.0, 6.4.40, 7.2.0, 7.3.0, 7.1.0, 5.4.52, 6.1.0, 6.2.0, 6.3.0 | [Symfony is a PHP framework for web and console applications and a...  <br>  <br>](https://avd.aquasec.com/nvd/cve-2026-45305)[https://avd.aquasec.com/nvd/cve-2026-45305](https://avd.aquasec.com/nvd/cve-2026-45305) |                                                                                                                                  |

	1)La cve avec le score CVSS le plus élevé est : CVE-2021-46743 avec un score CVSS de 9.1


| **Rang** | **CVE**        | **Bibliothèque**        | **Score CVSS** | **Sévérité** |
| -------- | -------------- | ----------------------- | -------------- | ------------ |
| 1        | CVE-2021-46743 | firebase/php-jwt        | 9.1            | Critique     |
| 2        | CVE-2026-45063 | symfony/security-http   | 9.1            | Critique     |
| 3        | CVE-2026-45077 | symfony/monolog-bridge  | 8.6            | Élevée       |
| 4        | CVE-2026-48489 | symfony/security-http   | 7.5            | Élevée       |
| 5        | CVE-2026-45133 | symfony/yaml            | 7.5            | Élevée       |
| 6        | CVE-2026-45304 | symfony/yaml            | 7.5            | Élevée       |
| 7        | CVE-2026-45305 | symfony/yaml            | 7.5            | Élevée       |
| 8        | CVE-2026-45073 | symfony/cache           | 7.3            | Élevée       |
| 9        | CVE-2025-64500 | symfony/http-foundation | 7.3            | Élevée       |
| 10       | CVE-2026-69246 | guzzlehttp/guzzle       | 7.2            | Élevée       |

	2)Oui le code utilise cette bibliothèque de façon risquée, indépendamment même de cette CVE. En effet, le Jetons "API" (JWT) est signés avec un secret faible, codé en dur, et sans expiration donc il est facilement rejouable, cassable hors-ligne, ou forgeable.
 

	3)Oui comme on peut le voir dans le premier tableau, il y a un correctif pour chaque ’une des CVE.

## Étape 5 : 

la commande a détecté le fichier src/Controller/Lab/A05MisconfigController.php à la ligne 13, trivy a détécté la clé d'acces AWS : 

11   // ATTENTION : clé codée en dur, volontairement laissée ici pour la démo Trivy --scanners secret  
12   // (voir Cours Trivy - TP2). Format valide d'une clé d'accès AWS, mais fictive.  
13 [ // AWS_ACCESS_KEY_ID=********************

## Étape 6 : 

	1) Le nombre total de CVE trouvées sur l'image est baucoup plus élevé que sur le simple scan `fs` de l'Étape 4, tellement que le powershell ne pouvais pas afficher toutes les CVE.

	2) En tentant de reconstruire l'image aujourd'hui, je n'ai pas rencontré d'erreur. voila ce que la commande m'a donné : 

	docker build -f docker/Dockerfile -t poc-symfony-owasp-cve:1.0 .  
[+] Building 19.5s (17/17) FINISHED                                                         docker:desktop-linux  
 => [internal] load build definition from Dockerfile                                                        0.1s => => transferring dockerfile: 1.64kB                                                                      0.0s => [internal] load metadata for docker.io/library/composer:2                                               1.3s => [internal] load metadata for docker.io/library/php:8.1.1-apache                                         1.3s => [internal] load .dockerignore                                                                           0.1s => => transferring context: 2B                                                                             0.0s => [stage-0  1/10] FROM docker.io/library/php:8.1.1-apache@sha256:456a0a47453ee517495f82cf334325c771845fc  0.1s => => resolve docker.io/library/php:8.1.1-apache@sha256:456a0a47453ee517495f82cf334325c771845fc45d1dc5209  0.1s => [internal] load build context                                                                           0.6s => => transferring context: 501.19kB                                                                       0.5s => FROM docker.io/library/composer:2@sha256:aaeab4b6b031e0a88efb907f0f26b563532a644fc2f4ea0d000ecf8658f7a  0.1s => => resolve docker.io/library/composer:2@sha256:aaeab4b6b031e0a88efb907f0f26b563532a644fc2f4ea0d000ecf8  0.1s => CACHED [stage-0  2/10] RUN apt-get update && apt-get install -y --no-install-recommends         libzip  0.0s => CACHED [stage-0  3/10] RUN {         echo 'display_errors = On';         echo 'expose_php = On';        0.0s => CACHED [stage-0  4/10] COPY --from=composer:2 /usr/bin/composer /usr/bin/composer                       0.0s => CACHED [stage-0  5/10] WORKDIR /var/www/html                                                            0.0s => CACHED [stage-0  6/10] COPY composer.json composer.lock symfony.lock ./                                 0.0s => CACHED [stage-0  7/10] RUN composer install --no-dev --no-scripts --no-interaction --optimize-autoload  0.0s => [stage-0  8/10] COPY . .                                                                                2.3s => [stage-0  9/10] COPY docker/apache/000-default.conf /etc/apache2/sites-available/000-default.conf       0.3s => [stage-0 10/10] RUN mkdir -p var public/uploads     && chmod -R 777 var public/uploads     && compose  10.2s => exporting to image                                                                                      3.5s => => exporting layers                                                                                     1.6s   
 => => exporting manifest sha256:30a53624efc6eab86640953010ca0004f09e0e056bfaa4d3dcc9c64a86be6e2b           0.0s  
 => => exporting config sha256:fd40f8a9edd5f1ade3ee82c27e2914c7ae8213386f90d826314cec896a5ea0e6             0.0s => => exporting attestation manifest sha256:d1d9f2d05fd3766172595b51af6fb303dccf41e1fbd0d54dc5106dfcfbdb5  0.1s   
 => => exporting manifest list sha256:c60908558ec05cbaa0644164a863f68a6eddf7d8f0a49e0380d6c0c4195a4c34      0.1s  
 => => naming to docker.io/library/poc-symfony-owasp-cve:1.0                                                0.0s => => unpacking to docker.io/library/poc-symfony-owasp-cve:1.0                                             1.5s

## Étape 7 : 

	1) la faillle AVD-DS-0002 est une faille relié aux droits des utilisateurs. (un utilisateur lambda peut avoir acces à des informations pour le devlopeur). dans /lab/a05, on peut voir que peu importe avec quel compte on est connécté, on a acces au phpinfo, au profiler et à l'API JSON (qui renvoie tout les mots de passes et identifiants de tout les utilisateurs)

	2) la faille recommande d'ajouter un moniteur aux conteneurs pour pouvoir les surveiller.

## Étape 8 : 


