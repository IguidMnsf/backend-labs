# LAB4 — Sécuriser l'API avec Spring Security et JWT

Dans ce laboratoire, on repart de l'API produits du LAB3 et on la protège. Une personne crée un compte **USER**, se connecte avec son mot de passe, reçoit un jeton JWT temporaire, puis le joint à ses requêtes. Les accès d'administration sont réservés au rôle **ADMIN**.

Le parcours d'une requête protégée est : **Postman → filtre JWT → identité et rôle → règle d'accès → contrôleur → réponse**. L'API est stateless : chaque requête apporte son jeton et le serveur ne stocke pas de session HTTP.

## Ce que contient le projet

- Java 21, Spring Boot, Spring Security, Spring Data JPA, MySQL et JJWT.
- Inscription et connexion avec réponses JSON et validation des champs.
- Mots de passe stockés avec BCrypt, jamais en clair.
- JWT signé, expirant après une durée configurable et vérifié par filtre.
- Rôle `USER` attribué automatiquement lors de toute inscription publique.
- Création initiale d'un ADMIN activée uniquement avec des paramètres d'environnement locaux.
- Accès en lecture aux produits et catégories pour USER/ADMIN ; modifications réservées à ADMIN.
- Commandes créées par un USER, visibles uniquement par leur propriétaire ; ADMIN peut consulter et supprimer toutes les commandes.
- Erreurs `401 Unauthorized` (non connecté/jeton invalide) et `403 Forbidden` (rôle insuffisant).
- CORS local restreint, CSRF désactivé pour l'API Bearer stateless.
- Collection Postman, tests H2 et diagrammes Mermaid dans `docs\uml`.

> **Règle essentielle :** le formulaire public d'inscription n'accepte pas le rôle. Même si quelqu'un ajoute `"role":"ADMIN"` à son JSON, le compte créé reste `USER`. Le compte ADMIN est configuré localement par la personne qui démarre le serveur; ses identifiants ne sont pas inclus dans GitHub.

## 1. Prérequis

Installe :

- JDK 21 ;
- Maven 3.6 ou supérieur ;
- Docker Desktop et son moteur démarré, ou MySQL 8 installé ;
- Postman Desktop.

Vérifie Java et Maven dans PowerShell :

```powershell
java -version
mvn -version
```

Il faut que Java indique la version 21. Si `java -version` affiche Java 8 et si `winget` est disponible, tu peux installer les outils ainsi :

```powershell
winget install --id EclipseAdoptium.Temurin.21.JDK -e
winget install --id Apache.Maven -e
```

Ferme PowerShell, rouvre-le et vérifie de nouveau les versions. Si plusieurs versions de Java sont installées, règle le `PATH` Windows pour utiliser le JDK 21.

## 2. Ouvrir le dossier du LAB4 en local

Sur cet ordinateur, le dépôt est ici :

```text
C:\Users\monsi\.copilot\repos\backend-labs
```

Dans PowerShell, mets-le à jour et entre dans le projet :

```powershell
Set-Location "$HOME\.copilot\repos\backend-labs"
git pull
Set-Location .\lab4-spring-security
Get-Location
```

Le chemin affiché doit finir par `backend-labs\lab4-spring-security`. Si tu n'as pas encore téléchargé le dépôt, clone-le une seule fois :

```powershell
Set-Location $HOME
git clone https://github.com/IguidMnsf/backend-labs.git
Set-Location .\backend-labs\lab4-spring-security
```

Les commandes Maven, Docker Compose et les exemples SQL ci-dessous sont à lancer dans `lab4-spring-security`.

## 3. Démarrer MySQL

### Méthode recommandée : Docker Desktop

Ouvre Docker Desktop et attends que son moteur ait démarré. Dans PowerShell, depuis le dossier `lab4-spring-security`, lance :

```powershell
docker compose up -d mysql
docker compose ps
docker compose logs mysql
```

Attends que le conteneur `lab4-mysql` soit en état `healthy`. Docker crée la base **`lab4_db`**. Ce LAB4 utilise volontairement sa propre base, distincte de celle du LAB3.

Pour se connecter à la base avec le client MySQL fourni dans Docker :

```powershell
docker compose exec mysql mysql -uroot -plab4pass lab4_db
```

Tu peux quitter l'invite MySQL avec `exit`. Pour ne pas supprimer tes données, ne lance pas `docker compose down -v`.

### MySQL installé localement

1. Démarre MySQL Server.
2. Crée la base `lab4_db` en exécutant `src\main\resources\db\01-create-database.sql` dans MySQL Workbench.
3. Configure les variables dans la même fenêtre PowerShell qui lancera Spring Boot :

```powershell
$env:DB_URL = "jdbc:mysql://localhost:3306/lab4_db?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=UTC"
$env:DB_USERNAME = "root"
$env:DB_PASSWORD = Read-Host "Mot de passe MySQL"
```

La table `users` est générée par JPA au démarrage. Tu peux aussi exécuter `src\main\resources\db\03-create-users-table.sql` après avoir créé la base.

## 4. Préparer une clé JWT et le compte ADMIN

Le secret JWT doit être privé, aléatoire, base64 et contenir au moins 32 octets. On le crée dans PowerShell sans l'inscrire dans le code ni dans Git :

```powershell
$bytes = New-Object byte[] 32
$rng = [System.Security.Cryptography.RandomNumberGenerator]::Create()
$rng.GetBytes($bytes)
$env:JWT_SECRET_BASE64 = [Convert]::ToBase64String($bytes)
$rng.Dispose()
```

Active ensuite la création locale du compte ADMIN. Choisis ton propre mot de passe (12 à 72 caractères). Quand PowerShell le demande, écris-le et appuie sur Entrée :

```powershell
$env:BOOTSTRAP_ADMIN_ENABLED = "true"
$env:BOOTSTRAP_ADMIN_USERNAME = "lab-admin"
$env:BOOTSTRAP_ADMIN_EMAIL = "lab-admin@example.test"
$env:BOOTSTRAP_ADMIN_PASSWORD = Read-Host "Choisis le mot de passe du compte ADMIN"
```

Ces variables ne sont disponibles que dans cette fenêtre PowerShell. C'est volontaire : elles ne sont ni enregistrées dans le dépôt, ni partagées avec GitHub. Au premier lancement, l'application crée le compte ADMIN et chiffre son mot de passe avec BCrypt. Si ce nom d'utilisateur existe déjà, elle ne remplace pas son mot de passe.

**Important :** saisis le même identifiant et mot de passe ADMIN dans les variables Postman indiquées à l'étape 7. Ne publie pas une capture montrant la clé JWT ou le mot de passe.

## 5. Compiler et démarrer l'API

Dans la fenêtre qui contient les variables JWT et ADMIN, toujours depuis `lab4-spring-security` :

```powershell
mvn clean test
mvn spring-boot:run
```

La première commande lance les tests isolés sur H2 et ne requiert pas MySQL. La seconde démarre l'API branchée à MySQL. Laisse ce terminal ouvert : le serveur doit afficher son message de démarrage et écoute sur `http://localhost:8080`.

Dans un second terminal ou dans le navigateur, vérifie que l'API répond :

```powershell
curl.exe -i http://localhost:8080/api/products
```

Sans jeton, cette route doit répondre **`401 Unauthorized`**. C'est un résultat attendu : l'API est bien protégée. Pour arrêter Spring Boot, clique dans le terminal du serveur puis appuie sur `Ctrl+C`.

Si tu arrêtes puis redémarres l'application dans la même fenêtre, les variables JWT et ADMIN restent définies. Si tu ouvres un nouveau terminal, il faut les définir de nouveau. Si tu régénères la clé JWT, les jetons déjà reçus deviennent invalides : reconnecte-toi.

## 6. Les requêtes disponibles et les rôles

| Méthode | URL | Accès |
|---|---|---|
| POST | `/api/auth/register` | Public : crée toujours un compte USER |
| POST | `/api/auth/login` | Public : vérifie les identifiants et renvoie un JWT |
| GET | `/api/products`, `/api/products/{id}`, `/api/products/search` | USER ou ADMIN |
| POST, PUT, DELETE | `/api/products...` | ADMIN |
| GET | `/api/categories...` | USER ou ADMIN |
| POST, PUT, DELETE | `/api/categories...` | ADMIN ; suppression refusée si la catégorie contient des produits |
| POST | `/api/orders` | USER ; la commande appartient au compte connecté |
| GET | `/api/orders`, `/api/orders/{id}` | USER voit ses commandes ; ADMIN voit toutes les commandes |
| DELETE | `/api/orders/{id}` | ADMIN |

Une requête privée doit porter l'en-tête :

```http
Authorization: Bearer <jeton reçu lors de la connexion>
```

Le jeton expire après une heure par défaut. Le rôle n'est pas accepté depuis le client et l'accès aux commandes est contrôlé aussi par propriétaire, pas seulement par identifiant.

## 7. Postman expliqué pas à pas

Postman sert à envoyer une requête à l'API et à afficher son statut et son JSON. **Postman ne démarre pas MySQL et ne démarre pas l'API** : garde Docker Desktop et le terminal `mvn spring-boot:run` ouverts.

### Importer la collection

1. Ouvre **Postman Desktop**.
2. Clique **Import** → **Files**.
3. Sélectionne `C:\Users\monsi\.copilot\repos\backend-labs\lab4-spring-security\postman\LAB4-Auth.postman_collection.json`.
4. Dans **Collections**, ouvre **LAB4 - Authentification JWT et rôles**.
5. Ouvre le menu `...` de la collection puis **Edit** ou **Variables**.
6. Dans **Current value** de `adminUsername`, saisis le nom configuré dans PowerShell (par défaut `lab-admin`).
7. Dans **Current value** de `adminPassword`, saisis le mot de passe ADMIN choisi lors du démarrage. Garde cette valeur dans Postman seulement et ne la capture pas à l'écran.
8. Laisse `baseUrl` à `http://localhost:8080`. La collection créera le nom unique du compte USER et mémorisera les jetons après connexion.

### Exécuter le parcours dans l'ordre

1. Ouvre **1. Créer un compte USER** puis clique **Send**. Résultat attendu : `201 Created`, avec `"role": "USER"`.
2. Ouvre **2. Connexion USER et récupération du JWT**, puis clique **Send**. Résultat attendu : `200 OK`. Le script enregistre le jeton dans `userToken`.
3. Ouvre **3. Connexion ADMIN et récupération du JWT**, puis clique **Send**. Le script enregistre le jeton ADMIN dans `adminToken`.
4. Envoie **4. Refuser les produits sans jeton** : `401 Unauthorized` est attendu.
5. Envoie **5. Créer une catégorie (ADMIN)**, puis **6. Créer un produit (ADMIN)**. Le `201 Created` confirme l'accès ADMIN ; les identifiants sont mémorisés automatiquement.
6. Vérifie les règles catégories avec **7. ADMIN modifie une catégorie** (`200`), puis **8. USER ne peut pas modifier une catégorie** (`403`).
7. Envoie **9. USER peut consulter les produits** : la réponse doit être `200 OK`.
8. Envoie **10. USER ne peut pas créer un produit** : `403 Forbidden` est attendu. Le compte est connecté, mais n'a pas le rôle ADMIN.
9. Envoie **11. USER crée une commande**, puis **12. USER voit ses commandes**. La commande est associée au propriétaire USER.
10. Envoie **13. ADMIN voit toutes les commandes**.
11. Termine par **14. USER ne peut pas supprimer une commande** (`403`), **15. ADMIN supprime une commande** (`204`), **16. ADMIN ne supprime pas une catégorie non vide** (`409`) et **17. Refuser un JWT invalide** (`401`).

Dans la réponse en bas de Postman, **Body** montre le JSON, le badge en haut à droite montre le statut (`200`, `401`, etc.) et **Test Results** montre les assertions. Les erreurs `401` et `403` de la collection sont intentionnelles : elles démontrent les protections.

Si le login ADMIN échoue, vérifie que tu as démarré l'application avec le bon mot de passe ADMIN, que `adminUsername`/`adminPassword` sont corrects dans Postman et que le conteneur MySQL est `healthy`. Si Postman ne joint pas `localhost`, vérifie d'abord `http://localhost:8080/api/products` dans ton navigateur, puis utilise Postman Desktop.

## 8. Tests automatiques

Depuis `lab4-spring-security`, exécute :

```powershell
mvn test
```

Les tests utilisent une base H2 temporaire. Ils vérifient l'inscription avec rôle USER, le hachage BCrypt, le login, les jetons manquants/invalides, les réponses 401/403, les accès ADMIN et l'isolation des commandes par utilisateur.

## 9. Captures d'écran à préparer

Utilise `Win + Shift + S`, sélectionne la zone, clique la notification puis **Enregistrer sous**. Range les captures, par exemple dans `Documents\Captures-Labs\LAB4`. Masque toujours les mots de passe, clés et jetons.

**Captures de démonstration locale :**

1. PowerShell avec `docker compose ps` montrant `lab4-mysql` `healthy`.
2. Terminal serveur après le démarrage réussi de Spring Boot (sans afficher les variables d'environnement).
3. Postman sur **1. Créer un compte USER**, avec le statut `201` et le rôle USER visible.
4. Postman sur **2. Connexion USER**, avec `200` et `tokenType: Bearer`; cache la valeur du jeton avant la capture.
5. Postman sur **8. USER ne peut pas créer un produit**, avec le statut `403`.
6. Postman sur **9. USER crée une commande** ou **10. USER voit ses commandes**, avec `201`/`200`.
7. Facultatif : PowerShell après `mvn test` avec `BUILD SUCCESS`.

**Captures du projet GitHub :**

1. Ouvre [les fichiers du LAB4 sur GitHub](https://github.com/IguidMnsf/backend-labs/tree/main/lab4-spring-security) et capture le dossier.
2. Capture le README affiché par GitHub.
3. Dans **Actions**, ouvre un run récent et capture le job `lab4-spring-security` terminé avec succès.
4. Facultatif : affiche la collection Postman ou un diagramme dans `docs/uml`.

GitHub héberge le code et exécute les tests. L'API n'est pas hébergée sur Internet par ce dépôt : les requêtes Postman `localhost:8080` sont servies par ton ordinateur.

## 10. CORS, CSRF et secrets

- La politique CORS de démonstration n'autorise que `http://localhost:3000` et `http://127.0.0.1:5500`, avec les méthodes API utiles. Modifie cette liste si ton frontend local utilise une autre adresse.
- CSRF est désactivé parce que l'API reçoit un jeton Bearer dans l'en-tête et n'utilise pas d'authentification par cookie de navigateur. Si l'authentification est un jour déplacée vers des cookies, cette décision devra être revue.
- La clé JWT et les identifiants ADMIN viennent des variables d'environnement locales. Ne les écris pas dans le code, dans Git, dans Postman partagé ou dans une capture.
- Les fichiers `docs/uml/auth-sequence.md` et `docs/uml/protected-request.md` contiennent les diagrammes de séquence.

## Dépannage

**`mvn` n'est pas reconnu** : installe Maven et rouvre PowerShell. **Java 8 apparaît** : installe/configure le JDK 21 avant de lancer le projet.

**Erreur de connexion MySQL** : démarre Docker Desktop, exécute `docker compose ps`, et confirme que l'application pointe vers `lab4_db`. Ce labo ne doit pas utiliser la base du LAB3.

**Erreur `JWT_SECRET_BASE64`** : génère la clé avec les commandes de l'étape 4 dans le même terminal que `mvn spring-boot:run`.

**Réponse 401** : reconnecte-toi dans Postman et vérifie que la collection utilise le jeton enregistré pour le bon compte. Un jeton expire ou est invalidé si la clé change.

**Réponse 403** : le jeton est valide, mais l'action est réservée à l'autre rôle. Par exemple, la création d'un produit exige ADMIN.

**La collection échoue au compte existant** : chaque exécution crée automatiquement un nom USER unique. N'envoie pas deux fois **1. Créer un compte USER** avant de refaire les deux connexions avec le nom sauvegardé.

**Arrêter MySQL** : depuis ce dossier, exécute `docker compose down`. Cette commande conserve le volume et les comptes de test.

## Résultat attendu

À la fin du LAB4, une personne peut créer un compte USER, recevoir un JWT après authentification, consulter l'API selon son rôle et créer uniquement ses propres commandes. Les opérations d'administration nécessitent un compte ADMIN créé localement de manière contrôlée.
