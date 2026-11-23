# LAB2 — API REST de produits avec Spring Boot

API Java 21 / Spring Boot 3 construite selon les couches **Controller → Service → Repository**. Les données sont conservées dans une `ConcurrentHashMap` pendant l'exécution : elles disparaissent au redémarrage, car ce laboratoire ne configure pas encore de base de données.

Le projet implémente aussi les exercices du sujet : recherche insensible à la casse par nom, exception métier lorsque la quantité dépasse 1000 et DTOs Java `record`.

## Prérequis

- JDK 21
- Maven 3.6 ou supérieur
- Postman (facultatif, `curl` peut être utilisé à la place)

Vérifier les outils :

```bash
java -version
mvn -version
```

## Lancer l'API

Depuis le dossier `lab2-springboot` :

```bash
mvn spring-boot:run
```

Le serveur démarre sur `http://localhost:8080`. L'API est vide au démarrage.

## Exécuter les tests automatisés

```bash
mvn test
```

Les tests couvrent la création/lecture, la recherche sans distinction de casse, la modification/suppression, les erreurs de validation, la quantité maximale et la ressource absente.

## Utiliser la collection Postman

1. Importer `postman/LAB2-Product-API.postman_collection.json`.
2. Démarrer l'API.
3. Ouvrir la collection et lancer **1. Créer un produit** en premier.
4. Le script de cette requête sauvegarde son identifiant dans `productId`.
5. Exécuter ensuite les requêtes de lecture, recherche, mise à jour et suppression. **La suppression est à lancer en dernier**, car les autres opérations utilisent le même produit.

L'environnement de la collection utilise `baseUrl=http://localhost:8080` et `productId=1`. Le script de création remplace l'identifiant par celui retourné par l'API.

## Endpoints

| Méthode | URL | Résultat |
|---|---|---|
| POST | `/api/products` | Crée un produit, statut `201 Created` |
| GET | `/api/products` | Liste tous les produits |
| GET | `/api/products/{id}` | Renvoie un produit ou `404 Not Found` |
| GET | `/api/products/search?name=laptop` | Recherche partielle sans distinction de casse |
| PUT | `/api/products/{id}` | Remplace les champs du produit, statut `200 OK` |
| DELETE | `/api/products/{id}` | Supprime le produit, statut `204 No Content` |

Exemple de corps JSON pour POST et PUT :

```json
{
  "name": "Ordinateur Portable",
  "description": "Laptop 15 pouces",
  "price": 1200.50,
  "quantity": 10
}
```

`name` doit être présent, `price` doit être strictement positif et `quantity` doit être positive ou nulle et ne pas dépasser 1000. Les erreurs de validation renvoient `400 Bad Request` avec les champs concernés. Une quantité supérieure à 1000 est traitée par `InvalidQuantityException`. Un identifiant inconnu renvoie `404 Not Found`.

## Tester avec curl dans PowerShell

L'API peut également être appelée sans Postman depuis une deuxième fenêtre PowerShell. Les formes JSON entre apostrophes sont compatibles avec PowerShell :

```powershell
curl.exe -i -X POST http://localhost:8080/api/products -H "Content-Type: application/json" -d '{"name":"Ordinateur Portable","description":"Laptop 15 pouces","price":1200.50,"quantity":10}'
curl.exe -i http://localhost:8080/api/products
curl.exe -i "http://localhost:8080/api/products/search?name=laptop"
```

La réponse de création contient l'`id`. Remplace `1` ci-dessous par cet identifiant si nécessaire :

```powershell
$id = 1
curl.exe -i "http://localhost:8080/api/products/$id"
curl.exe -i -X PUT "http://localhost:8080/api/products/$id" -H "Content-Type: application/json" -d '{"name":"Ordinateur Portable Pro","description":"Laptop amélioré","price":1300.00,"quantity":8}'
curl.exe -i -X DELETE "http://localhost:8080/api/products/$id"
```

## Arborescence

```text
src/main/java/com/example/productapi/
├── controller/       # Routes HTTP
├── dto/              # ProductRequest et ProductResponse
├── exception/        # Exceptions métier et handler global
├── model/            # Modèle Product
├── repository/       # Stockage temporaire thread-safe
└── service/          # Logique métier et recherche
```

La réponse de création est bien HTTP `201`; elle ajoute également l'en-tête `Location`. Les opérations d'écriture ne sont pas persistées au redémarrage : l'ajout d'une base de données est l'objet du LAB3.

## Démarrage local avec PowerShell

Cloner le dépôt dans le dossier utilisateur puis accéder au projet :

```powershell
Set-Location $HOME
git clone https://github.com/IguidMnsf/backend-labs.git
Set-Location .\backend-labs\lab2-springboot
```

Les commandes supposent un clone dans `$HOME\backend-labs`. Si le dépôt est enregistré ailleurs, remplacer ce chemin par son emplacement réel. Pour un dépôt déjà cloné :

```powershell
Set-Location "$HOME\backend-labs"
git pull --ff-only
Set-Location .\lab2-springboot
Get-Location
```

Vérifie que Java 21 et Maven sont installés :

```powershell
java -version
mvn -version
```

La première commande doit afficher Java 21. Pour installer le JDK 21 avec `winget` :

```powershell
winget install --id EclipseAdoptium.Temurin.21.JDK -e
```

Si `mvn` n'est pas reconnu, installer Apache Maven depuis [la page officielle de téléchargement](https://maven.apache.org/download.cgi), ajouter le dossier `bin` de Maven au `PATH` Windows, puis ouvrir une nouvelle fenêtre PowerShell. Si Chocolatey est déjà installé, l'installation peut aussi se faire avec `choco install maven -y`.

Après l'installation, fermer puis rouvrir PowerShell et vérifier `java -version` et `mvn -version`. En présence de plusieurs versions de Java, configurer le `PATH` Windows pour sélectionner le JDK 21. Depuis le dossier du projet, démarrer le serveur :

```powershell
mvn spring-boot:run
```

Garder cette fenêtre ouverte. Lorsque Spring Boot affiche `Started ...Application`, l'API est disponible à `http://localhost:8080`. `localhost` désigne l'ordinateur sur lequel le serveur s'exécute. Pour arrêter le serveur, utiliser `Ctrl+C` dans cette fenêtre.

## Postman expliqué pas à pas

Postman sert à envoyer une requête à l'API et à lire sa réponse. Postman ne démarre pas le serveur : la fenêtre PowerShell contenant `mvn spring-boot:run` doit rester ouverte.

1. Ouvre Postman Desktop et clique sur **Import** → **Files**.
2. Dans le clone local, ouvrir `lab2-springboot\postman` et importer `LAB2-Product-API.postman_collection.json`.
3. Dans **Collections**, ouvre **LAB2 - API Produits Spring Boot**. L'adresse enregistrée est `http://localhost:8080`.
4. Clique sur **1. Créer un produit**, puis **Send**.
5. Vérifie le statut `201 Created` et l'onglet **Body** en bas : le produit apparaît sous forme de JSON. Son identifiant est mémorisé par Postman.
6. Exécute ensuite les requêtes de liste, recherche, lecture et mise à jour. Pour lancer plusieurs requêtes dans l'ordre, clique sur **Run** de la collection.
7. Exécute la requête de suppression en dernier. Les onglets **Body** et **Test Results** montrent respectivement la réponse et les tests.

Si Postman dans un navigateur n'accède pas à `localhost`, utiliser Postman Desktop ou sélectionner l'agent Desktop/local. Vérifier également que le serveur répond à `http://localhost:8080/api/products`.

## Captures d'écran à préparer

Utiliser `Win + Shift + S`, sélectionner la zone, puis choisir **Enregistrer sous** dans l'outil Capture d'écran. Enregistrer les images dans un dossier dédié, par exemple `Documents\Captures-Labs\LAB2`.

**Captures locales :**

1. PowerShell dans `lab2-springboot`, avec le serveur démarré.
2. Postman après **1. Créer un produit**, avec l'adresse localhost, le statut `201 Created` et le JSON visibles.
3. Postman avec les **Test Results** ou le résultat du Run de la collection.
4. Facultatif : PowerShell après `mvn test`, indiquant `BUILD SUCCESS`.

**Captures du projet publié sur GitHub :**

1. Ouvre [les fichiers LAB2 sur GitHub](https://github.com/IguidMnsf/backend-labs/tree/main/lab2-springboot).
2. Capture cette page puis le README ouvert sur GitHub.
3. Ouvre **Actions** du dépôt et capture un run récent où le job `lab2-springboot` est terminé avec succès.

La page GitHub et le workflow montrent que le code est publié et vérifié. Il ne s'agit pas d'un déploiement Internet : l'API fonctionne localement.
