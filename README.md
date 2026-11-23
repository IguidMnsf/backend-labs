# Mes travaux pratiques — Développement backend

Bienvenue dans mon espace de travaux pratiques en développement backend. Chaque dossier correspond à un laboratoire complet, avec le code, son propre guide de démarrage et les fichiers utiles pour le tester.

## Les laboratoires

| Travail pratique | Contenu | Démarrage |
|---|---|---|
| [TP1 — API REST et Postman](tp1-postman/README.md) | Requêtes HTTP, collection Postman et rapport | Importer la collection dans Postman |
| [LAB2 — API produits Spring Boot](lab2-springboot/README.md) | Java 21, API REST en mémoire, DTO, validation et tests | `cd lab2-springboot` puis `mvn spring-boot:run` |
| [LAB3 — API Spring Data JPA et MySQL](lab3-springboot/README.md) | Java 21, JPA, MySQL, catégories, produits, commandes et tests | `cd lab3-springboot` puis démarrer MySQL et Spring Boot |
| [LAB4 — Authentification JWT et rôles](lab4-spring-security/README.md) | Spring Security, JWT, BCrypt, rôles USER/ADMIN et commandes privées | `cd lab4-spring-security` puis démarrer MySQL et Spring Boot |
| [LAB5 — API produits Node.js](lab5-node-product-api/README.md) | Express, CRUD, validation, Swagger et Postman | `cd lab5-node-product-api` puis `npm install` et `npm run dev` |

## Cloner tous les labs

Dans PowerShell ou un terminal :

```powershell
git clone https://github.com/IguidMnsf/backend-labs.git
cd backend-labs
```

Le clone utilisé sur cet ordinateur se trouve dans :

```text
C:\Users\monsi\.copilot\repos\backend-labs
```

Dans PowerShell, tu peux y accéder avec un chemin indépendant du nom de compte :

```powershell
Set-Location "$HOME\.copilot\repos\backend-labs"
```

Les étapes d'installation, les commandes PowerShell, l'utilisation de Postman et les captures à préparer sont détaillées dans le README de chaque laboratoire.

## Important : GitHub et application locale

Le dépôt GitHub héberge le **code source** et les guides. Les LAB2, LAB3 et LAB5 sont exécutés sur ton ordinateur : publier leurs fichiers sur GitHub ne rend pas automatiquement l'API accessible sur Internet. Pour tester ces trois API dans Postman, démarre le serveur local puis utilise l'adresse indiquée dans le guide, par exemple `http://localhost:8080` ou `http://localhost:3000`.

TP1 utilise le service public DummyJSON et ne demande pas de serveur local.

## Captures d'écran pour présenter les labs

Après avoir terminé le test local, appuie sur `Win + Shift + S`, sélectionne la zone à capturer, puis clique sur la notification de capture pour ouvrir l'outil Capture d'écran et **Enregistrer sous**. Range les images dans un dossier de ton choix, par exemple `Documents\Captures-Labs`. Chaque README propose les captures locales et GitHub propres à son laboratoire. Pour LAB2, LAB3 et LAB5, la page **Actions** montre les tests automatiques terminés avec succès ; ce n'est pas un déploiement public de l'API.

## Vérifications automatiques

GitHub Actions exécute les tests du LAB2, du LAB3 et du LAB4 avec Java 21, et ceux du LAB5 avec Node.js 20. Le TP1 est constitué de livrables Postman et d'un rapport.

Les API des LAB2 et LAB5 conservent leurs données en mémoire uniquement : les données de test reviennent à leur état initial quand le serveur redémarre. Le LAB3 utilise MySQL pour conserver les données.
