# 🔐 Documentation Technique : Système d'Authentification Web

Ce document détaille l'architecture, le flux de données et la configuration du module de sécurité de l'API Amethral. Le système repose sur une approche hybride combinant **OAuth2** pour l'identification et **JWT** pour la gestion de session.

---

## 1. Architecture Globale

Le module d'authentification a pour but de vérifier l'identité de l'utilisateur via des fournisseurs de confiance (Google, Discord) et de délivrer un jeton d'accès sécurisé pour l'API.

### Diagramme de Flux
Le schéma ci-dessous illustre les interactions entre le Client (Vue.js), l'API .NET, les Providers OAuth et la base PostgreSQL.

<img width="1024" height="559" alt="Architecture Authentification Amethral" src="https://github.com/user-attachments/assets/97d4a510-f973-457a-83e6-ada2fa4c9801" />

---

## 2. Principes Techniques

### 2.1. Double Mécanisme
L'API utilise deux schémas d'authentification ASP.NET Core en parallèle :
1.  **Cookie Authentication (Interne) :** Utilisé uniquement de manière temporaire lors du "handshake" OAuth pour stocker les claims renvoyés par Google/Discord avant la génération du JWT.
2.  **JWT Bearer (Externe) :** Le standard utilisé pour sécuriser tous les endpoints de l'API. Le token est signé en **HMACSHA256**.

### 2.2. Workflow de Connexion
1.  **Initiation :** Le frontend appelle un endpoint de challenge (ex: `/api/auth/login/google`).
2.  **Redirection :** L'API redirige l'utilisateur vers la page de consentement du fournisseur.
3.  **Callback :**
    * Le fournisseur renvoie l'utilisateur vers l'API avec un code.
    * L'API échange ce code contre les informations de l'utilisateur (Email, ID, Avatar).
4.  **Résolution d'Identité (`AuthService`) :**
    * **Scénario A (Connexion) :** Le couple `Provider` + `ProviderKey` existe en base → *Authentification réussie.*
    * **Scénario B (Liaison) :** L'email existe mais pas ce provider → *Création du lien `UserOAuth`.*
    * **Scénario C (Inscription) :** L'email est inconnu → *Création automatique du `User` et du `UserOAuth`.*
5.  **Génération JWT :** Un token contenant les claims essentiels (`sub`, `email`, `username`) est généré et renvoyé au client.

---

## 3. Modèle de Données

Les données sont structurées pour permettre à un utilisateur unique de posséder plusieurs méthodes de connexion.

### Entités (Entity Framework)

* **`User`** (Compte Principal)
    * `Id` (GUID)
    * `Email` (Unique)
    * `Username`
* **`UserOAuth`** (Méthode de connexion)
    * `ProviderName` (ex: "Google")
    * `ProviderKey` (ID unique chez Google)
    * `UserId` (Clé étrangère vers `User`)

> **Règle d'intégrité :** Un même compte Google ne peut pas être lié à plusieurs utilisateurs différents.

---

## 4. Configuration Technique

Pour que l'authentification fonctionne, le fichier `appsettings.json` (ou les User Secrets) doit contenir les clés suivantes.

### Configuration (`appsettings.json`)

```json
{
  "JwtSettings": {
    "Key": "VOTRE_SECRET_KEY_DOIT_ETRE_LONGUE_ET_SECURISEE",
    "Issuer": "AmethralAPI",
    "Audience": "AmethralClient",
    "DurationInDays": 7
  },
  "Authentication": {
    "Google": {
      "ClientId": "PLACEHOLDER_GOOGLE_CLIENT_ID",
      "ClientSecret": "PLACEHOLDER_GOOGLE_CLIENT_SECRET"
    },
    "Discord": {
      "ClientId": "PLACEHOLDER_DISCORD_CLIENT_ID",
      "ClientSecret": "PLACEHOLDER_DISCORD_CLIENT_SECRET"
    }
  }
}
