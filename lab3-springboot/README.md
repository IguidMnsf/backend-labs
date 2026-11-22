# LAB3 — Spring Data JPA, Hibernate et MySQL

Ce laboratoire fait évoluer l’API du LAB2 : les produits ne disparaissent plus à l’arrêt du serveur, ils sont enregistrés dans MySQL. Le projet gère aussi les catégories et les commandes, avec les relations entre les entités, des DTO pour les réponses JSON et des tests automatisés.

Le parcours d’une requête est : **Controller → Service → Repository → Hibernate/JPA → MySQL**.

## Ce qui est inclus

- Spring Boot 3, Java 21 et Maven.
- Entités `Product`, `Category`, `CustomerOrder` et `OrderItem`.
- Relations `Category 1—N Product`, `CustomerOrder 1—N OrderItem` et `Product 1—N OrderItem`.
- Repositories Spring Data JPA, dont une recherche JPQL par catégorie.
- Services transactionnels et DTO Java `record`, sans exposer les entités JPA en JSON.
- API REST pour les catégories, produits, pagination et commandes.
- Validation des requêtes, erreurs JSON cohérentes et protection des produits utilisés par une commande.
- MySQL 8 avec Docker Compose, ou configuration d’un MySQL installé localement.
- Profil de test H2 : les tests n’ont pas besoin d’un serveur MySQL.
- Collection Postman et scripts SQL d’initialisation/exemple.

## 1. Prérequis

Installe :

- JDK 21 ;
- Maven 3.6 ou supérieur ;
- Postman, si tu veux exécuter la collection ;
- Docker Desktop pour la méthode MySQL la plus simple, **ou** MySQL Server et MySQL Workbench.

Dans PowerShell, vérifie Java et Maven :

```powershell
java -version
mvn -version
```

La sortie de Java doit indiquer la version 21. Si `mvn` n’est pas reconnu, installe Apache Maven ou utilise Maven intégré à IntelliJ IDEA.

## 2. Télécharger le dépôt et entrer dans le dossier du LAB3

Pour télécharger les TP1, LAB2 et LAB5 en même temps :

```powershell
cd $HOME
git clone https://github.com/IguidMnsf/backend-labs.git
cd backend-labs
```

Si tu as déjà le dépôt localement :

```powershell
cd $HOME\backend-labs
git pull
```

Puis va dans le projet Spring Boot :

```powershell
cd lab3-springboot
```

**Important :** toutes les commandes Maven et Docker Compose ci-dessous sont à lancer depuis `backend-labs\lab3-springboot`.

## 3. Installer MySQL

### Option A — Docker Compose (recommandée)

Démarre MySQL :

```powershell
docker compose up -d mysql
```

Vérifie qu’il est prêt :

```powershell
docker compose ps
docker compose logs mysql
```

Le service crée automatiquement la base `lab3_db`. Les paramètres locaux sont déjà réglés dans `application.properties` :

- hôte : `localhost:3306` ;
- base : `lab3_db` ;
- utilisateur : `root` ;
- mot de passe de développement : `lab3pass`.

### Option B — MySQL installé sur l’ordinateur

1. Démarre MySQL Server depuis les services Windows ou MySQL Workbench.
2. Ouvre MySQL Workbench et connecte-toi au serveur local.
3. Ouvre `src\main\resources\db\01-create-database.sql` et exécute-le.
4. Configure ton utilisateur et ton mot de passe dans le terminal avant de démarrer Spring Boot :

```powershell
$env:DB_USERNAME = "root"
$env:DB_PASSWORD = "ton_mot_de_passe_mysql"
```

Si MySQL n’écoute pas sur le port `3306`, définis aussi `DB_URL` :

```powershell
$env:DB_URL = "jdbc:mysql://localhost:3307/lab3_db?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=UTC"
```

Ces variables ne sont définies que dans la fenêtre PowerShell courante. Il n’y a pas de vrai mot de passe MySQL stocké dans Git.

## 4. Installer et lancer l’API

Depuis `lab3-springboot` :

```powershell
mvn clean install
mvn spring-boot:run
```

Le serveur écoute sur `http://localhost:8080`. Hibernate crée ou met à jour les tables lors du premier démarrage (`ddl-auto=update`).

Tu peux vérifier les produits :

```powershell
curl.exe http://localhost:8080/api/products
```

Un résultat vide est normal tant que tu n’as pas créé de catégorie et de produit.

Dans une autre fenêtre PowerShell, tu peux aussi ouvrir MySQL :

```powershell
docker compose exec mysql mysql -uroot -plab3pass lab3_db
```

Puis vérifier les lignes persistées :

```sql
SELECT * FROM categories;
SELECT * FROM products;
SELECT * FROM customer_orders;
SELECT * FROM order_items;
```

Quitte le client MySQL avec `exit`. Pour arrêter l’API, reviens au terminal du serveur et utilise `Ctrl+C`.

## 5. Créer une catégorie et un produit

Une catégorie doit exister avant la création du produit, car chaque produit appartient à une catégorie.

### Créer la catégorie

```powershell
curl.exe -i -X POST http://localhost:8080/api/categories -H "Content-Type: application/json" -d '{"name":"Électronique"}'
```

La réponse est `201 Created`. Note l’`id` renvoyé : il sera utilisé comme `categoryId`.

### Créer le produit

Remplace `1` par l’identifiant réel de la catégorie :

```powershell
curl.exe -i -X POST http://localhost:8080/api/products -H "Content-Type: application/json" -d '{"name":"Smartphone","price":699.99,"categoryId":1}'
```

La réponse contient le produit, son identifiant et sa catégorie (`categoryId`, `categoryName`). Les entités JPA ne sont pas retournées directement, ce qui évite les boucles JSON entre catégorie et produits.

## 6. Endpoints disponibles

| Méthode | URL | Résultat |
|---|---|---|
| POST | `/api/categories` | Créer une catégorie, `201` |
| GET | `/api/categories` | Lister les catégories |
| GET | `/api/categories/{id}` | Lire une catégorie |
| GET | `/api/categories/{id}/products` | Lister les produits d’une catégorie |
| POST | `/api/products` | Créer un produit lié à une catégorie, `201` |
| GET | `/api/products?page=0&size=10&sort=id&direction=asc` | Lister les produits avec pagination et tri |
| GET | `/api/products/{id}` | Lire un produit |
| GET | `/api/products/search?name=phone` | Rechercher par nom, sans tenir compte de la casse |
| PUT | `/api/products/{id}` | Modifier le nom, le prix ou la catégorie |
| DELETE | `/api/products/{id}` | Supprimer un produit non utilisé dans une commande, `204` |
| POST | `/api/orders` | Créer une commande avec une ou plusieurs lignes, `201` |
| GET | `/api/orders` | Lister les commandes |
| GET | `/api/orders/{id}` | Lire une commande et son total |

Un produit associé à une commande ne peut pas être supprimé : l’API renvoie `409 Conflict` pour préserver l’intégrité des références.

### Créer une commande

Remplace `1` par l’identifiant d’un produit existant :

```powershell
curl.exe -i -X POST http://localhost:8080/api/orders -H "Content-Type: application/json" -d '{"items":[{"productId":1,"quantity":2}]}'
```

La réponse détaille les lignes de commande, le prix unitaire mémorisé et le total. La commande et ses lignes sont créées dans une seule transaction. Le prix de la ligne est copié depuis le produit au moment de la commande afin de préserver le prix d’achat historique.

## 7. Tester avec Postman

1. Démarre MySQL et l’API comme indiqué plus haut.
2. Dans Postman, clique **Import**.
3. Importe `postman/LAB3-API.postman_collection.json`.
4. Vérifie la variable `baseUrl` : `http://localhost:8080`.
5. Exécute les requêtes dans l’ordre : créer catégorie, créer produit, lister/lire/rechercher, modifier, créer commande, lire commande.
6. La collection mémorise automatiquement les identifiants générés.
7. Les requêtes de validation et de conflits peuvent être lancées ensuite.

L’identifiant généré dépend des données déjà présentes : les scripts Postman le capturent au lieu de supposer qu’il vaut toujours `1`.

## 8. Exécuter les tests automatisés

Les tests utilisent une base H2 temporaire avec le profil `test`, donc MySQL n’a pas besoin d’être démarré :

```powershell
mvn test
```

Les tests couvrent les relations catégorie-produit, la pagination, la recherche, la mise à jour, les commandes transactionnelles, la validation et les erreurs `404`/`409`.

## 9. Données d’exemple SQL

Hibernate crée les tables à partir des entités. Si tu veux ajouter le jeu d’exemple du sujet, après le démarrage initial du serveur, ouvre un client MySQL et exécute :

```powershell
Get-Content -Raw src\main\resources\db\02-sample-data.sql | docker compose exec -T mysql mysql -uroot -plab3pass lab3_db
```

Dans MySQL Workbench, tu peux aussi ouvrir le fichier `src\main\resources\db\02-sample-data.sql` puis l’exécuter.

Les données d’exemple sont facultatives : la collection Postman crée elle-même ses données de test.

## 10. Comprendre les relations et le JSON

- `Product` possède une relation `ManyToOne` vers `Category`, chargée en `LAZY`.
- `Category` expose une collection `OneToMany` en lecture interne, sans cascade de suppression dangereuse.
- `CustomerOrder` possède plusieurs `OrderItem` ; la suppression des lignes suit le cycle de vie de la commande.
- Chaque `OrderItem` fait référence à un produit et mémorise quantité et prix unitaire.
- Les contrôleurs retournent des DTO, pas les entités JPA : cela évite les cycles de sérialisation et le chargement accidentel de relations.
- Les modifications métier sont encadrées par `@Transactional`.

## Dépannage

**`Access denied for user`** : vérifie `DB_USERNAME` et `DB_PASSWORD`, puis ouvre un nouveau terminal si tu as modifié ces variables.

**`Communications link failure`** : assure-toi que MySQL tourne et écoute sur le port configuré. Avec Docker, vérifie `docker compose ps`.

**La base n’existe pas** : exécute `01-create-database.sql` ou recrée le conteneur après avoir corrigé sa configuration.

**Port 8080 déjà occupé** : utilise un autre port dans la même fenêtre PowerShell :

```powershell
$env:SERVER_PORT = "8081"
mvn spring-boot:run
```

**Produit ou catégorie introuvable** : crée d’abord la catégorie, puis le produit, et utilise les identifiants retournés par l’API.

**Ne vois pas de données après redémarrage** : contrairement au LAB2, ce projet utilise MySQL. Vérifie que tu te connectes à la même base. Les tests H2, en revanche, sont temporaires et isolés.

**Réinitialiser la base Docker** : la commande suivante supprime définitivement le volume MySQL du labo et toutes ses données locales. Ne l’exécute que si tu veux recommencer à zéro :

```powershell
docker compose down -v
docker compose up -d mysql
```

## Résultat attendu

À la fin du LAB3, tu as une API Spring Boot qui conserve les catégories, produits et commandes dans MySQL, un modèle relationnel cohérent, des réponses JSON sans cycle, et une collection Postman pour tester le parcours complet.
