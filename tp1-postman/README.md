# TP1 — API REST et Postman

Ce dossier contient les livrables du laboratoire :

- `TP1-API-REST-Postman.postman_collection.json` : collection Postman v2.1 avec les requêtes, les tests et les dossiers du TP.
- `Demo-API-Env.postman_environment.json` : environnement Postman prêt à importer.
- `rapport-tp1.md` : rapport des observations, également fourni en PDF.
- `rapport-tp1.pdf` : rapport à remettre.

## Importer dans Postman

1. Dans Postman, choisissez **Import** puis importez les deux fichiers `.json`.
2. Dans la liste des environnements, sélectionnez **Demo API Env**.
3. Ouvrez la collection **LAB1 - Gestion Produits REST**.
4. Exécutez les requêtes. Les scripts affichent les assertions dans **Test Results**.
5. Pour exécuter toute la collection, utilisez le **Collection Runner**. L'ordre des requêtes est organisé par dossier.

L'environnement contient `baseUrl`, `productId` et `createdProductId`. La requête de création initialise `createdProductId` à partir de la réponse.

## Résultats à interpréter

DummyJSON est une API de démonstration : ses opérations d'ajout, de modification et de suppression sont simulées et ne constituent pas une persistance durable. Le temps de réponse peut dépasser 500 ms selon le réseau ; ce test mesure aussi la connexion utilisée pour l'exercice.

Les codes HTTP observés lors de la préparation du rapport sont consignés dans `rapport-tp1.md`. Les résultats peuvent varier dans le temps ou selon le service distant.

## Ouvrir les fichiers locaux avec PowerShell

Pour accéder au TP1 dans un clone créé sous le dossier utilisateur :

```powershell
Set-Location "$HOME\backend-labs"
git pull --ff-only
Set-Location .\tp1-postman
Get-Location
```

Le dernier chemin doit finir par `backend-labs\tp1-postman`. Si le dépôt n'est pas encore cloné :

```powershell
Set-Location $HOME
git clone https://github.com/IguidMnsf/backend-labs.git
Set-Location .\backend-labs\tp1-postman
```

## Postman expliqué simplement

Postman envoie une requête HTTP à un site ou à une API, puis affiche la réponse. Dans ce TP, les requêtes sont envoyées au service public DummyJSON. Aucun serveur local n'est à démarrer ; une connexion Internet est nécessaire.

1. Ouvre Postman et clique **Import** → **Files**.
2. Dans le clone local, ouvrir `tp1-postman` et importer `TP1-API-REST-Postman.postman_collection.json` ainsi que `Demo-API-Env.postman_environment.json`.
3. Dans la liste déroulante des environnements (en haut à droite), sélectionne **Demo API Env**.
4. Dans **Collections**, ouvre **LAB1 - Gestion Produits REST**, puis déplie le dossier **Lecture (GET)**.
5. Clique sur **1. Lister les 5 premiers produits**, puis **Send**.
6. En bas de Postman, choisis **Body** pour voir les données JSON. Le statut attendu est `200 OK`. Ouvre **Test Results** pour lire les vérifications automatiques.
7. Clique sur d'autres requêtes pour tester lecture, recherche, création et modification. La collection organise les exercices par dossier.
8. Pour exécuter l'ensemble, clique sur **Run** de la collection, puis lance la séquence proposée.

Dans l'environnement **Demo API Env**, `baseUrl` doit être `https://dummyjson.com`. Ne change pas cette adresse en `localhost` : contrairement aux LAB2/LAB3/LAB5, le TP1 utilise un service public et n'a pas d'API locale.

DummyJSON simule certaines écritures. Une création ou une suppression peut renvoyer un succès sans que la donnée reste enregistrée ensuite.

## Captures d'écran à remettre

Utiliser `Win + Shift + S`, sélectionner la zone, puis choisir **Enregistrer sous** dans la notification de capture. Enregistrer les images dans un dossier dédié, par exemple `Documents\Captures-Labs\TP1`.

1. Dans Postman, afficher **1. Lister les 5 premiers produits**, l'URL `https://dummyjson.com`, le statut `200 OK` et le JSON de réponse.
2. Afficher la même requête avec **Test Results** ouvert et les assertions visibles.
3. Afficher **Demo API Env** sélectionné et la variable `baseUrl`.
4. Facultatif : afficher les résultats du Runner après exécution de la collection.
5. Après publication, ouvrir [le dossier TP1 sur GitHub](https://github.com/IguidMnsf/backend-labs/tree/main/tp1-postman) et capturer les livrables et le README.

Le TP1 n'a pas de serveur à déployer ni de workflow Actions. Les captures GitHub montrent que la collection et le rapport sont publiés ; les captures Postman montrent les requêtes effectivement exécutées.
