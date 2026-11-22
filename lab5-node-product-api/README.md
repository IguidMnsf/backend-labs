# LAB5 — API REST de produits avec Node.js et Express

Ce projet réalise une API REST complète pour gérer des produits. Il suit une architecture en couches et permet de créer, lire, modifier partiellement, remplacer et supprimer des produits.

Les données sont conservées **uniquement en mémoire**. Elles reviennent à leurs trois valeurs initiales au redémarrage du serveur.

## Ce qui est livré

- API Express avec modules ECMAScript.
- Architecture Routes, Controllers, Services, Repositories, Models et Middlewares.
- Validation des entrées avec `express-validator`.
- Gestion centralisée des réponses d'erreur.
- Documentation interactive Swagger/OpenAPI.
- Collection Postman prête à importer.
- Tests automatisés avec Node.js.
- Workflow GitHub Actions pour vérifier le projet.

## Prérequis

Installez les outils suivants :

- **Node.js 18 ou supérieur** ; Node.js 20 est recommandé.
- **npm** fourni avec Node.js.
- **Postman** ou `curl`.
- **Visual Studio Code** ou un autre éditeur.

Sous Windows, vérifiez l'installation depuis PowerShell :

```powershell
node --version
npm --version
```

Si une de ces commandes n'est pas reconnue, installez Node.js depuis https://nodejs.org/, puis redémarrez le terminal.

## Télécharger le projet

### Depuis GitHub

Dans PowerShell :

```powershell
git clone https://github.com/IguidMnsf/backend-labs.git
cd backend-labs
```

Si le dépôt existe déjà sur ton ordinateur, mets-le à jour :

```powershell
Set-Location "$HOME\.copilot\repos\backend-labs"
git pull
```

Dans ce projet, les fichiers du LAB5 se trouvent dans :

```text
lab5-node-product-api
```

## Installation des dépendances

Depuis la racine du dépôt, ouvre le dossier du laboratoire :

```powershell
Set-Location .\lab5-node-product-api
npm install
```

La commande `npm install` installe :

- `express` : serveur HTTP et routage.
- `dotenv` : chargement du fichier `.env`.
- `express-validator` : validation des entrées.
- `swagger-ui-express` : interface Swagger.
- `swagger-autogen` : génération du fichier OpenAPI.
- `supertest` : tests HTTP automatisés.

Pour reproduire exactement les versions enregistrées :

```powershell
npm ci
```

## Configuration du port

Le serveur utilise le port `3000` par défaut.

Tu peux créer un fichier `.env` à partir de l'exemple :

```powershell
Copy-Item .env.example .env
```

Le contenu est :

```dotenv
PORT=3000
```

Pour utiliser un autre port, modifie uniquement la valeur dans `.env`, par exemple `PORT=4000`. Toutes les adresses locales devront alors utiliser `4000`.

## Lancer le serveur

Depuis `lab5-node-product-api` :

```powershell
npm run dev
```

Le mode `dev` utilise `node --watch` : Express redémarre automatiquement après une modification.

Tu peux aussi démarrer normalement :

```powershell
npm start
```

Lorsque le serveur démarre, tu dois voir des messages indiquant les adresses de l'API et de Swagger.

Vérifie rapidement dans le navigateur :

- `http://localhost:3000/`
- `http://localhost:3000/api/products`
- `http://localhost:3000/api-docs`
- `http://localhost:3000/api-docs.json`

Pour arrêter le serveur, appuie sur `Ctrl+C` dans le terminal.

## Générer Swagger

La documentation est générée automatiquement au démarrage. Tu peux aussi la régénérer explicitement :

```powershell
npm run swagger
```

Cette commande crée `src/docs/swagger-output.json`. Swagger UI lit ensuite ce fichier dans `http://localhost:3000/api-docs`.

## Endpoints disponibles

| Méthode | URL | Action | Résultat |
|---|---|---|---|
| GET | `/` | Vérifier que le serveur fonctionne | `200` |
| GET | `/api/products` | Lister les produits | `200` |
| GET | `/api/products/:id` | Lire un produit | `200` ou `404` |
| POST | `/api/products` | Créer un produit | `201` |
| PUT | `/api/products/:id` | Remplacer complètement un produit | `200` ou `404` |
| PATCH | `/api/products/:id` | Modifier seulement certains champs | `200` ou `404` |
| DELETE | `/api/products/:id` | Supprimer un produit | `204` ou `404` |

Corps JSON attendu pour POST et PUT :

```json
{
  "name": "Écran 24 pouces",
  "price": 199.99,
  "stock": 10
}
```

Corps JSON pour PATCH :

```json
{
  "stock": 20
}
```

Les règles principales sont :

- `name` : chaîne non vide.
- `price` : nombre positif ou nul.
- `stock` : entier positif ou nul.
- `id` : entier supérieur ou égal à 1.

## Tester avec Postman

1. Lance l'API avec `npm run dev`.
2. Ouvre Postman.
3. Choisis **Import**.
4. Importe `postman/LAB5-Product-API.postman_collection.json`.
5. Vérifie que la variable `baseUrl` vaut `http://localhost:3000`.
6. Lance d'abord **3. Créer un produit**.
7. La requête sauvegarde automatiquement l'identifiant créé dans `productId`.
8. Lance ensuite les autres requêtes dans l'ordre.
9. Lance **10. Supprimer le produit créé** en dernier.

Tu peux aussi utiliser directement Swagger UI :

`http://localhost:3000/api-docs`

## Tester avec curl sous Windows

Le dépôt indique `curl.exe` pour éviter les alias PowerShell.

### Vérifier le serveur

```powershell
curl.exe http://localhost:3000/
```

### Lister les produits

```powershell
curl.exe http://localhost:3000/api/products
```

### Lire le produit 1

```powershell
curl.exe http://localhost:3000/api/products/1
```

### Créer un produit

```powershell
curl.exe -X POST http://localhost:3000/api/products -H "Content-Type: application/json" -d '{"name":"Tapis","price":10,"stock":5}'
```

### Remplacer complètement le produit 1

```powershell
curl.exe -X PUT http://localhost:3000/api/products/1 -H "Content-Type: application/json" -d '{"name":"Clavier Gamer","price":80,"stock":10}'
```

### Modifier seulement le stock du produit 1

```powershell
curl.exe -X PATCH http://localhost:3000/api/products/1 -H "Content-Type: application/json" -d '{"stock":20}'
```

### Supprimer le produit 3

```powershell
curl.exe -X DELETE http://localhost:3000/api/products/3
```

## Tester les erreurs

### Identifiant invalide

```powershell
curl.exe -i http://localhost:3000/api/products/1abc
```

Réponse attendue : `400 Bad Request`.

### Produit inexistant

```powershell
curl.exe -i http://localhost:3000/api/products/999
```

Réponse attendue : `404 Not Found`.

### Corps invalide

```powershell
curl.exe -i -X POST http://localhost:3000/api/products -H "Content-Type: application/json" -d '{"name":"","price":-1,"stock":0}'
```

Réponse attendue : `400 Bad Request` avec les détails des champs invalides.

### Route inconnue

```powershell
curl.exe -i http://localhost:3000/api/unknown
```

Réponse attendue : `404 Not Found` avec le message « Route non trouvée ».

## Exécuter les tests automatisés

Depuis `lab5-node-product-api` :

```powershell
npm test
```

Les tests vérifient notamment :

- la route de bienvenue ;
- le cycle CRUD complet ;
- l'identifiant invalide ;
- le produit inexistant ;
- les données invalides ;
- les routes inconnues ;
- la disponibilité de Swagger JSON.

## Architecture du projet

```text
lab5-node-product-api/
├── src/
│   ├── controllers/
│   │   └── product.controller.js
│   ├── docs/
│   │   └── swagger-output.json
│   ├── middlewares/
│   │   ├── error.middleware.js
│   │   └── validator.middleware.js
│   ├── models/
│   │   └── product.model.js
│   ├── repositories/
│   │   └── product.repository.js
│   ├── routes/
│   │   └── product.routes.js
│   ├── services/
│   │   └── product.service.js
│   └── server.js
├── postman/
│   └── LAB5-Product-API.postman_collection.json
├── test/
│   └── product-api.test.js
├── swagger.js
├── package.json
└── README.md
```

Le fonctionnement suit ce chemin :

**requête HTTP → route → middleware de validation → controller → service → repository → tableau en mémoire → réponse JSON**

Les erreurs remontent vers le middleware centralisé `error.middleware.js`, ce qui évite de dupliquer la gestion des exceptions dans chaque endpoint.

## Dépannage

### `npm` n'est pas reconnu

1. Installe Node.js depuis https://nodejs.org/.
2. Ferme tous les terminaux ouverts.
3. Ouvre un nouveau PowerShell.
4. Vérifie `node --version` et `npm --version`.

### Le port 3000 est déjà utilisé

Modifie `PORT` dans `.env` :

```dotenv
PORT=4000
```

Puis relance `npm run dev` et utilise `http://localhost:4000`.

### Swagger ne s'affiche pas

Exécute :

```powershell
npm run swagger
npm run dev
```

Puis ouvre `http://localhost:3000/api-docs` ou vérifie `src/docs/swagger-output.json`.

### Les modifications disparaissent après redémarrage

C'est normal pour le LAB5 : le repository utilise un tableau JavaScript en mémoire. L'ajout d'une vraie base de données fera partie d'un laboratoire suivant.

## Résultat attendu

À la fin du LAB5, tu disposes d'une API Node.js exécutable, d'une structure modulaire, d'une validation robuste, d'une gestion centralisée des erreurs et d'une documentation Swagger interactive. Tu peux présenter directement l'API, les tests, la collection Postman et cette documentation sur GitHub.

## Accéder au projet local et le lancer avec PowerShell

Sur cet ordinateur, le dossier est :

```text
C:\Users\monsi\.copilot\repos\backend-labs\lab5-node-product-api
```

Ouvre PowerShell, mets le dépôt à jour, entre dans le LAB5 et installe les dépendances :

```powershell
Set-Location "$HOME\.copilot\repos\backend-labs"
git pull
Set-Location .\lab5-node-product-api
Get-Location
npm ci
```

Le chemin affiché doit finir par `backend-labs\lab5-node-product-api`. Si tu n'as pas le dépôt localement, clone-le :

```powershell
Set-Location $HOME
git clone https://github.com/IguidMnsf/backend-labs.git
Set-Location .\backend-labs\lab5-node-product-api
npm ci
```

Vérifie Node.js et npm, puis démarre le serveur :

```powershell
node --version
npm --version
npm run dev
```

Garde le terminal ouvert. Quand le message du serveur apparaît, essaie `http://localhost:3000/api/products` dans ton navigateur. Pour arrêter le serveur, appuie sur `Ctrl+C`.

## Postman expliqué pas à pas

Postman envoie une requête au serveur qui tourne dans le terminal. Il ne lance pas `npm run dev`.

1. Ouvre Postman Desktop, clique **Import** → **Files**.
2. Importe `C:\Users\monsi\.copilot\repos\backend-labs\lab5-node-product-api\postman\LAB5-Product-API.postman_collection.json`.
3. Dans **Collections**, ouvre **LAB5 - API Produits Node.js Express** et vérifie que `baseUrl` vaut `http://localhost:3000`.
4. Dans un terminal PowerShell séparé, démarre `npm run dev` et laisse-le ouvert.
5. Dans Postman, ouvre **1. Vérifier le serveur**, puis clique **Send**. Le résultat attendu est `200 OK`.
6. Ouvre **3. Créer un produit** et clique **Send**. Vérifie `201 Created` et le produit JSON dans l'onglet **Body**. L'identifiant est enregistré automatiquement.
7. Envoie ensuite les requêtes de liste, lecture, PUT et PATCH. Les onglets **Body** et **Test Results** montrent respectivement la réponse et les assertions.
8. Les requêtes d'erreur peuvent s'exécuter ensuite. Envoie **10. Supprimer le produit créé** tout à la fin.
9. Pour exécuter toutes les requêtes dans l'ordre, clique **Run** sur la collection.

`localhost` signifie « mon ordinateur » : cette URL ne fonctionne que quand le serveur est démarré localement. Si Postman Web ne peut pas accéder à cette adresse, choisis l'agent Desktop/local ou utilise l'application Postman Desktop.

## Captures d'écran pour le professeur

Appuie sur `Win + Shift + S`, sélectionne la zone, clique la notification de capture puis **Enregistrer sous**. Enregistre les images, par exemple dans `Documents\Captures-Labs\LAB5`.

**Captures en local :**

1. PowerShell dans `lab5-node-product-api` avec le serveur en cours d'exécution.
2. Postman sur **1. Vérifier le serveur**, avec `localhost:3000` et `200 OK`.
3. Postman sur **3. Créer un produit**, avec `201 Created` et le JSON visible.
4. Swagger dans le navigateur à `http://localhost:3000/api-docs`.
5. Facultatif : PowerShell après `npm test` avec tous les tests réussis.

**Captures du projet publié sur GitHub :**

1. Ouvre [les fichiers LAB5 sur GitHub](https://github.com/IguidMnsf/backend-labs/tree/main/lab5-node-product-api).
2. Capture le README et la liste des fichiers du dossier.
3. Dans **Actions**, ouvre un run récent et capture le job `lab5-node` terminé avec succès.

La publication GitHub et la réussite des tests ne mettent pas l'API en ligne. L'API est exécutée sur ton PC et utilise `localhost:3000`.
