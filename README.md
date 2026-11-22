# Mes travaux pratiques — Développement backend

Bienvenue dans mon espace de travaux pratiques en développement backend. Chaque dossier correspond à un laboratoire complet, avec le code, son propre guide de démarrage et les fichiers utiles pour le tester.

## Les laboratoires

| Travail pratique | Contenu | Démarrage |
|---|---|---|
| [TP1 — API REST et Postman](tp1-postman/README.md) | Requêtes HTTP, collection Postman et rapport | Importer la collection dans Postman |
| [LAB2 — API produits Spring Boot](lab2-springboot/README.md) | Java 21, API REST en mémoire, DTO, validation et tests | `cd lab2-springboot` puis `mvn spring-boot:run` |
| [LAB5 — API produits Node.js](lab5-node-product-api/README.md) | Express, CRUD, validation, Swagger et Postman | `cd lab5-node-product-api` puis `npm install` et `npm run dev` |

## Cloner tous les labs

Dans PowerShell ou un terminal :

```powershell
git clone https://github.com/IguidMnsf/backend-labs.git
cd backend-labs
```

Les étapes d'installation, les commandes PowerShell, les tests et les URL locales sont détaillés dans le README de chaque laboratoire.

## Vérifications automatiques

GitHub Actions exécute les tests du LAB2 avec Java 21 et ceux du LAB5 avec Node.js 20. Le TP1 est constitué de livrables Postman et d'un rapport.

Les API des LAB2 et LAB5 conservent leurs données en mémoire uniquement : les données de test reviennent à leur état initial quand le serveur redémarre.
