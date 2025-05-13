# 📘 Documentation Technique - Base de Données (Projet DevOps Arduino → C# → MySQL sur Azure)

## 🧩 1. Contexte du Projet

Ce projet met en œuvre un système de supervision environnementale basé sur des capteurs Arduino connectés à une application Windows Forms développée en C#. Les mesures collectées (CO2, humidité, température, luminosité, temps) sont transmises par trames réseau (TCP/IP) à l’application, puis stockées dans une base de données **MySQL hébergée sur Azure**.

Le projet suit une approche **DevOps** intégrant CI/CD, automatisation du déploiement, journalisation, et respecte les principes de **conformité RGPD**.

---

## 🗂️ 2. Objectifs de la base de données

- Centraliser les données issues des capteurs.
- Garantir la persistance, la traçabilité et la validité des mesures.
- Permettre l'analyse en temps réel et l’historique des mesures.
- Intégrer la base dans une architecture sécurisée et automatisée sur Azure.
- Appliquer les principes RGPD (sécurité, limitation, traçabilité).

---

## 🛠️ 3. Structure de la Base de Données

### 🔹 Table `capteurs`

| Colonne       | Type         | Contraintes                    | Description                          |
|---------------|--------------|--------------------------------|--------------------------------------|
| id_capteur    | INT          | PRIMARY KEY, AUTO_INCREMENT    | Identifiant unique du capteur       |
| type          | VARCHAR(50)  | NOT NULL                       | Type de capteur (CO2, TEMP, etc.)   |
| emplacement   | VARCHAR(100) | NULL                           | Emplacement physique ou logique     |
| description   | TEXT         | NULL                           | Détail facultatif sur le capteur    |

---

### 🔹 Table `mesures`

Chaque trame envoyée contient les types suivants : `temps`, `humidité`, `température`, `luminosité`, `CO2`.  
Chaque mesure est stockée indépendamment avec son type pour assurer la flexibilité et la compatibilité avec des futures extensions.

| Colonne       | Type         | Contraintes                    | Description                                   |
|---------------|--------------|--------------------------------|-----------------------------------------------|
| id_mesure     | INT          | PRIMARY KEY, AUTO_INCREMENT    | Identifiant unique de la mesure              |
| type_mesure   | VARCHAR(50)  | NOT NULL                       | `"temps"`, `"humidité"`, `"température"`, `"luminosité"`, `"CO2"` |
| valeur        | FLOAT        | NOT NULL                       | Valeur mesurée                               |
| horodatage    | DATETIME     | NOT NULL, DEFAULT CURRENT_TIMESTAMP | Date et heure de la mesure             |
| id_capteur    | INT          | FOREIGN KEY → capteurs(id_capteur) | Lien vers le capteur                        |

---

### 🔹 Table `logs_trames` (optionnelle)

| Colonne         | Type          | Description                                        |
|-----------------|---------------|----------------------------------------------------|
| id_log          | INT           | PRIMARY KEY, AUTO_INCREMENT                       |
| date_evenement  | DATETIME      | Timestamp de l’événement                          |
| evenement       | VARCHAR(100)  | Type d’événement (`insert`, `erreur`, `parsing`) |
| contenu_trame   | TEXT          | Trame brute reçue                                 |
| commentaire     | TEXT          | Informations ou erreur associée                   |

---

## 🧱 4. Déploiement et Intégration Azure

- Instance **Azure Database for MySQL**, hébergée en **France Central** (RGPD).
- Connexions sécurisées **SSL/TLS**.
- Configuration du pare-feu Azure : IP autorisées uniquement (serveur C# / CI).
- Stockage persistant et **sauvegardes automatiques**.
- Provisioning via **Azure CLI** ou **Terraform** (Infrastructure as Code).

---

## 🔐 5. Sécurité & RGPD

- Aucun traitement de données personnelles (mesures techniques uniquement).
- Conformité RGPD assurée par :
  - Chiffrement SSL.
  - Limitation des accès à la base (utilisateur avec privilèges restreints).
  - Politique de conservation (ex : suppression des mesures > 30 jours).
  - Journalisation (logs des trames erronées ou refusées).

---

## 🔄 6. Intégration DevOps

- Script SQL versionné (`init_db.sql`).
- Pipeline CI/CD avec :
  - Déploiement automatique de la base.
  - Tests de connectivité et d’insertion.
  - Lancement des scripts initiaux.
- Utilisation d’un gestionnaire de secrets (Azure Key Vault, GitLab CI).

---

## 📈 7. Cas d’usage et flux de données

1. L’opérateur démarre la surveillance.
2. L’Arduino envoie périodiquement des trames (via TCP/IP).
3. L’application C# :
   - Parse les trames.
   - Valide les valeurs.
   - Insère dans la base MySQL Azure.
4. Les graphiques sont mis à jour en temps réel.
5. Les erreurs sont loguées (si activé).

---

## ✅ 8. Conclusion

Cette base de données s’inscrit dans une architecture complète et conforme aux exigences DevOps et RGPD. Elle constitue un socle robuste pour la supervision en temps réel, avec une intégration cloud, une gestion sécurisée et des possibilités d’extension futures (analyse, alertes, dashboard...).

---

## 🧾 9. Script SQL `init_db.sql`

```sql
-- Création de la table capteurs
CREATE TABLE capteurs (
    id_capteur INT IDENTITY(1,1) PRIMARY KEY,
    type NVARCHAR(50) NOT NULL,
    emplacement NVARCHAR(100) NULL,
    description NVARCHAR(MAX) NULL
);

-- Création de la table mesures
CREATE TABLE mesures (
    id_mesure INT IDENTITY(1,1) PRIMARY KEY,
    type_mesure NVARCHAR(50) NOT NULL,
    valeur FLOAT NOT NULL,
    horodatage DATETIME2 NOT NULL DEFAULT SYSDATETIME(),
    id_capteur INT FOREIGN KEY REFERENCES capteurs(id_capteur)
);

-- Création de la table logs_trames (optionnelle)
CREATE TABLE logs_trames (
    id_log INT IDENTITY(1,1) PRIMARY KEY,
    date_evenement DATETIME2 NOT NULL DEFAULT SYSDATETIME(),
    evenement NVARCHAR(100) NOT NULL,
    contenu_trame NVARCHAR(MAX) NULL,
    commentaire NVARCHAR(MAX) NULL
);

```
