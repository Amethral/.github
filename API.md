# Documentation Technique — API Amethral

Bienvenue sur la documentation technique de l'API Amethral. Ce projet sert de backend pour Amethral, développé avec **ASP.NET Core** et **PostgreSQL**.

---

## 🔐 Système d'Authentification

Le module de sécurité est le cœur de l'API. Il implémente une **architecture hybride** permettant à la fois une authentification locale et une délégation via des fournisseurs tiers (OAuth2).

### Caractéristiques Principales
* **Protocole :** JWT (JSON Web Token) signé en HMACSHA256.
* **Fournisseurs supportés :** Google, Discord.
* **Gestion des identités :** Possibilité de lier plusieurs comptes externes (Google + Discord) à un seul compte utilisateur interne (`User` ↔ `UserOAuth`).

### Flux d'Authentification (Workflow)

1.  **Initiation :** Le client (Vue.js) demande une connexion (ex: `/oauth/google/login`).
2.  **Redirection :** L'API redirige l'utilisateur vers le provider (Google/Discord).
3.  **Callback & Validation :**
    * Au retour, l'API intercepte le code d'accès via un cookie temporaire.
    * Le service `AuthService` vérifie si l'email existe ou crée un nouveau `User`.
4.  **Délivrance du Token :** L'API génère un **JWT** contenant les `claims` (ID, email, username) valide pour 7 jours.
5.  **Sécurisation :** Ce token doit être envoyé en header `Authorization: Bearer <token>` pour accéder aux endpoints protégés.

### Modèle de Données & Sécurité
* **Base de données :** Séparation stricte entre l'identité (`Users`) et les méthodes de connexion (`UserOAuths`).
* **Intégrité :** Contraintes d'unicité sur les emails et les paires Provider/Key.
* **Configuration :** Les secrets (ClientIds, SecretKeys) sont injectés via `appsettings.json` ou les variables d'environnement (Azure KeyVault/UserSecrets en dev), jamais hardcodés.

### Architecture Visuelle
Le schéma ci-dessous illustre les interactions entre le Client, l'API .NET, le Provider OAuth et la base PostgreSQL.

<img width="1024" height="559" alt="image" src="https://github.com/user-attachments/assets/97d4a510-f973-457a-83e6-ada2fa4c9801" />

---

## 🛠 Stack Technique

* **Framework :** .NET 8 (ASP.NET Core Web API)
* **ORM :** Entity Framework Core
* **Base de données :** PostgreSQL
* **Documentation API :** Swagger / OpenAPI

## ⚙️ Installation & Configuration

### Pré-requis
1.  Cloner le dépôt.
2.  Disposer d'une instance PostgreSQL locale ou distante.

### Configuration (`appsettings.json`)
Assurez-vous de configurer les sections suivantes (ou d'utiliser les User Secrets) :

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Host=...;Database=mmorpg_auth;..."
  },
  "JwtSettings": {
    "Key": "VOTRE_CLE_SECRETE_TRES_LONGUE",
    "Issuer": "AmethralAPI",
    "Audience": "AmethralClient"
  },
  "Authentication": {
    "Google": { "ClientId": "...", "ClientSecret": "..." },
    "Discord": { "ClientId": "...", "ClientSecret": "..." }
  }
}
