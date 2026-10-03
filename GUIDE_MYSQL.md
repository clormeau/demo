# Guide MySQL pour débuter

MySQL est un système de gestion de bases de données relationnelles. Il permet de stocker des informations dans des tables et de les interroger avec le langage SQL.

## 1. Installer MySQL sous Windows

1. Téléchargez **MySQL Installer** depuis le site officiel : <https://dev.mysql.com/downloads/installer/>.
2. Lancez l'installation et choisissez **MySQL Server**. **MySQL Workbench** est facultatif : c'est une interface graphique pour administrer le serveur et exécuter des requêtes.
3. Pendant la configuration, conservez le port proposé (`3306`) sauf si une autre application l'utilise.
4. Choisissez un mot de passe pour le compte administrateur `root` et conservez-le dans un gestionnaire de mots de passe. Ne le mettez pas dans le code ou dans le dépôt.
5. Une fois le serveur démarré, ouvrez PowerShell et connectez-vous :

```powershell
mysql -u root -p
```

Saisissez le mot de passe lorsque MySQL le demande. Si `mysql` n'est pas reconnu, ajoutez le dossier `bin` de l'installation MySQL au `PATH`, ou ouvrez **MySQL Command Line Client** depuis le menu Démarrer.

## 2. Créer une base et une table

Les instructions SQL se terminent par un point-virgule. Les mots-clés ne sont pas sensibles à la casse.

```sql
CREATE DATABASE demo CHARACTER SET utf8mb4;
USE demo;

CREATE TABLE contacts (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nom VARCHAR(100) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE,
    cree_le TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

`utf8mb4` permet de stocker les caractères Unicode, y compris les accents et les emoji.

## 3. Ajouter, lire et modifier des données

```sql
INSERT INTO contacts (nom, email)
VALUES ('Ada Lovelace', 'ada@example.com');

SELECT id, nom, email, cree_le
FROM contacts
ORDER BY nom;

UPDATE contacts
SET email = 'ada.lovelace@example.com'
WHERE id = 1;
```

Vérifiez toujours la condition `WHERE` d'une requête `UPDATE` ou `DELETE`. Sans elle, toutes les lignes de la table peuvent être modifiées ou supprimées.

Pour supprimer uniquement ce contact :

```sql
DELETE FROM contacts
WHERE id = 1;
```

## 4. Créer un compte applicatif

Pour une application, préférez un compte dédié plutôt que le compte administrateur `root`. Dans cet exemple, remplacez le mot de passe et n'accordez les permissions qu'à la base nécessaire :

```sql
CREATE USER 'demo_app'@'localhost' IDENTIFIED BY 'remplacer-par-un-mot-de-passe-fort';
GRANT SELECT, INSERT, UPDATE, DELETE
ON demo.*
TO 'demo_app'@'localhost';
```

Évitez de publier le vrai mot de passe dans un dépôt. En production, utilisez un gestionnaire de secrets ou une variable d'environnement.

## 5. Se connecter depuis Python (facultatif)

Installez le connecteur officiel MySQL pour Python :

```powershell
py -m pip install mysql-connector-python
```

Exemple de connexion utilisant des variables d'environnement pour ne pas inscrire le mot de passe dans le script :

```python
import os
import mysql.connector

connection = mysql.connector.connect(
    host=os.getenv("MYSQL_HOST", "localhost"),
    user=os.getenv("MYSQL_USER", "demo_app"),
    password=os.environ["MYSQL_PASSWORD"],
    database="demo",
)

try:
    cursor = connection.cursor()
    cursor.execute(
        "SELECT id, nom, email FROM contacts WHERE email = %s",
        ("ada@example.com",),
    )
    for contact in cursor.fetchall():
        print(contact)
finally:
    connection.close()
```

Les paramètres `%s` évitent de construire une requête en concaténant directement des valeurs. Définissez `MYSQL_PASSWORD` dans l'environnement avant de lancer le script.

## 6. Sauvegarder et restaurer

Dans PowerShell, créez une sauvegarde de la base avec `mysqldump` :

```powershell
mysqldump -u demo_app -p demo > demo.sql
```

Pour restaurer cette sauvegarde dans une base existante :

```powershell
mysql -u demo_app -p demo < demo.sql
```

Une restauration peut remplacer des données existantes. Testez-la d'abord sur une base de test et vérifiez que les sauvegardes sont conservées en lieu sûr.

## Ressources

- [Documentation officielle MySQL](https://dev.mysql.com/doc/)
- [Téléchargements MySQL](https://dev.mysql.com/downloads/)
