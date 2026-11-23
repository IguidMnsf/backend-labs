# Travaux pratiques — Développement backend

Ce dépôt regroupe cinq travaux pratiques de développement backend. Chaque dossier contient le code source, un guide de démarrage et les fichiers nécessaires aux tests.

## Les laboratoires

| Travail pratique | Contenu | Démarrage |
|---|---|---|
| [TP1 — API REST et Postman](tp1-postman/README.md) | Requêtes HTTP, collection Postman et rapport | Importer la collection dans Postman |
| [LAB2 — API produits Spring Boot](lab2-springboot/README.md) | Java 21, API REST en mémoire, DTO, validation et tests | `cd lab2-springboot` puis `mvn spring-boot:run` |
| [LAB3 — API Spring Data JPA et MySQL](lab3-springboot/README.md) | Java 21, JPA, MySQL, catégories, produits, commandes et tests | `cd lab3-springboot` puis démarrer MySQL et Spring Boot |
| [LAB4 — Authentification JWT et rôles](lab4-spring-security/README.md) | Spring Security, JWT, BCrypt, rôles USER/ADMIN et commandes privées | `cd lab4-spring-security` puis démarrer MySQL et Spring Boot |
| [LAB5 — API produits Node.js](lab5-node-product-api/README.md) | Express, CRUD, validation, Swagger et Postman | `cd lab5-node-product-api` puis `npm ci` et `npm run dev` |

## Rapports et archives à remettre

Les rapports PDF et les archives ZIP prêtes à télécharger sont regroupés dans [Rapports et archives](livrables/README.md).

## Cloner le dépôt

Dans PowerShell, cloner le dépôt dans le dossier utilisateur :

```powershell
Set-Location $HOME
git clone https://github.com/IguidMnsf/backend-labs.git
Set-Location .\backend-labs
```

Pour mettre à jour un clone existant :

```powershell
Set-Location "$HOME\backend-labs"
git pull --ff-only
```

Les exemples supposent que le dépôt est dans `$HOME\backend-labs`. Si le clone se trouve ailleurs, remplacer ce chemin par son emplacement réel.

Les prérequis, commandes PowerShell, étapes Postman et captures à préparer sont détaillés dans le README de chaque laboratoire.

## En cas d'erreur : méthode de dépannage

Avant de modifier le code, vérifier ces quatre points dans l'ordre :

1. **Le bon dossier est ouvert** : dans PowerShell, exécuter `Get-Location`. Pour revenir au dossier du dépôt : `Set-Location "$HOME\backend-labs"`.
2. **Les outils sont disponibles** : vérifier `java -version` et `mvn -version` pour les LAB2 à LAB4, ou `node --version` et `npm --version` pour le LAB5.
3. **Les services nécessaires fonctionnent** : démarrer l'API dans une fenêtre PowerShell et la laisser ouverte ; pour les LAB3/LAB4, démarrer MySQL avant l'API.
4. **Postman vise le bon serveur** : choisir Postman Desktop, vérifier l'environnement, l'adresse `localhost` et l'ordre des requêtes. Le TP1 est l'exception : il utilise DummyJSON sur Internet, sans serveur local.

Après chaque correction, relancer uniquement l'étape qui avait échoué. Les guides ci-dessous donnent les messages fréquents et les commandes de vérification : [TP1](tp1-postman/README.md#dépannage), [LAB2](lab2-springboot/README.md#dépannage-pas-à-pas), [LAB3](lab3-springboot/README.md#dépannage-pas-à-pas), [LAB4](lab4-spring-security/README.md#dépannage-pas-à-pas), [LAB5](lab5-node-product-api/README.md#dépannage-pas-à-pas).

## Important : GitHub et application locale

Le dépôt GitHub héberge le **code source** et les guides. Les API des LAB2 à LAB5 s'exécutent localement : leur publication sur GitHub ne les rend pas automatiquement accessibles sur Internet. Pour utiliser Postman, démarrer l'API concernée puis employer l'adresse indiquée dans son guide, par exemple `http://localhost:8080` ou `http://localhost:3000`.

TP1 utilise le service public DummyJSON et ne demande pas de serveur local.

## Captures d'écran pour présenter les labs

Après chaque test local, utiliser `Win + Shift + S`, sélectionner la zone à capturer, puis choisir **Enregistrer sous** dans l'outil Capture d'écran. Classer les images dans un dossier dédié, par exemple `Documents\Captures-Labs`. Chaque README précise les captures locales et GitHub propres au laboratoire. La page **Actions** présente les tests automatiques ; elle ne constitue pas un déploiement public de l'API.

## Vérifications automatiques

GitHub Actions exécute les tests des LAB2, LAB3 et LAB4 avec Java 21, ainsi que ceux du LAB5 avec Node.js 20. Le TP1 comprend les livrables Postman et un rapport.

Les API des LAB2 et LAB5 conservent leurs données en mémoire uniquement : les données de test reviennent à leur état initial quand le serveur redémarre. Le LAB3 utilise MySQL pour conserver les données.
