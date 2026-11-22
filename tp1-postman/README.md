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

Sur cet ordinateur, le dépôt est dans `C:\Users\monsi\.copilot\repos\backend-labs`. Pour ouvrir le TP1 :

```powershell
Set-Location "$HOME\.copilot\repos\backend-labs"
git pull
Set-Location .\tp1-postman
Get-Location
```

Le dernier résultat doit finir par `backend-labs\tp1-postman`. Si tu n'as pas encore le dépôt, clone-le une seule fois :

```powershell
Set-Location $HOME
git clone https://github.com/IguidMnsf/backend-labs.git
Set-Location .\backend-labs\tp1-postman
```

## Postman expliqué simplement

Postman est une application qui envoie une requête HTTP à un site/API et affiche sa réponse. Dans ce TP, les requêtes vont sur Internet vers DummyJSON. **Tu n'as pas à démarrer de serveur dans PowerShell**, mais tu dois avoir une connexion Internet.

1. Ouvre Postman et clique **Import** → **Files**.
2. Dans `C:\Users\monsi\.copilot\repos\backend-labs\tp1-postman`, importe `TP1-API-REST-Postman.postman_collection.json` et `Demo-API-Env.postman_environment.json`.
3. Dans la liste déroulante des environnements (en haut à droite), sélectionne **Demo API Env**.
4. Dans **Collections**, ouvre **LAB1 - Gestion Produits REST**, puis déplie le dossier **Lecture (GET)**.
5. Clique sur **1. Lister les 5 premiers produits**, puis **Send**.
6. En bas de Postman, choisis **Body** pour voir les données JSON. Le statut attendu est `200 OK`. Ouvre **Test Results** pour lire les vérifications automatiques.
7. Clique sur d'autres requêtes pour tester lecture, recherche, création et modification. La collection organise les exercices par dossier.
8. Pour exécuter l'ensemble, clique sur **Run** de la collection, puis lance la séquence proposée.

Dans l'environnement **Demo API Env**, `baseUrl` doit être `https://dummyjson.com`. Ne change pas cette adresse en `localhost` : contrairement aux LAB2/LAB3/LAB5, le TP1 utilise un service public et n'a pas d'API locale.

DummyJSON simule certaines écritures. Une création ou une suppression peut renvoyer un succès sans que la donnée reste enregistrée ensuite.

## Captures d'écran à remettre

Appuie sur `Win + Shift + S`, sélectionne la zone, clique la notification de capture puis **Enregistrer sous**. Enregistre les images, par exemple dans `Documents\Captures-Labs\TP1`.

1. Capture Postman sur **1. Lister les 5 premiers produits** : on doit voir l'URL `https://dummyjson.com`, le statut `200 OK` et le JSON de réponse.
2. Capture la même requête avec **Test Results** ouvert et les assertions visibles.
3. Capture **Demo API Env** sélectionné et la variable `baseUrl`.
4. Facultatif : capture le Runner après exécution de la collection et ses résultats.
5. Après publication, ouvre [le dossier TP1 sur GitHub](https://github.com/IguidMnsf/backend-labs/tree/main/tp1-postman) et capture le dossier des livrables et le README affiché par GitHub.

Le TP1 n'a pas de serveur à déployer ni de workflow Actions. Les captures GitHub montrent que la collection et le rapport sont publiés ; les captures Postman montrent les requêtes effectivement exécutées.
