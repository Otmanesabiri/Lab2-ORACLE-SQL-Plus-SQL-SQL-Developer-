# Compte Rendu - TP1 : Installation et Gestion d'Oracle 21c XE sur Docker

**Préparé par :** Otmanesabiri  
**Date :** 17 Mars 2025  




```bash
# Vérifier l'état du conteneur 

docker ps -a | grep glsid2025

# Démarrer le conteneur

docker start glsid2025

# Attendre que Oracle soit complètement initialisé (peut prendre 2-5 minutes) 
docker logs -f glsid2025

```
![alt text](<Capture d’écran du 2025-04-06 17-51-28.png>)

### Se connecter à SQL*Plus :

```bash
docker exec -it glsid2025 sqlplus sys/Glsid2024-2025@XE as sysdba

```

![alt text](<Capture d’écran du 2025-04-06 17-51-46.png>)

### Localisation du fichier glogin.sql

```bash
docker exec -it glsid2025 bash -c "ls -la /opt/oracle/product/*/dbhome*/sqlplus/admin/glogin.sql"
```

![alt text](<Capture d’écran du 2025-04-06 17-59-52.png>)

### Visualiser le contenu de glogin.sql

```bash
docker exec -it glsid2025 cat /opt/oracle/product/21c/dbhomeXE/sqlplus/admin/glogin.sql
```

![alt text](<Capture d’écran du 2025-04-06 18-00-36.png>)

#### Personnalisation de SQL*Plus pour notre Travail

```bash
# Faire une copie de sauvegarde d'abord
docker exec -it glsid2025 cp /opt/oracle/product/21c/dbhomeXE/sqlplus/admin/glogin.sql /opt/oracle/product/21c/dbhomeXE/sqlplus/admin/glogin.sql.bak
```
![alt text](<Capture d’écran du 2025-04-06 18-02-34.png>)



```bash
# Ajouter vos personnalisations
docker exec -it glsid2025 bash -c "echo 'SET PAGESIZE 50' >> /opt/oracle/product/21c/dbhomeXE/sqlplus/admin/glogin.sql"
docker exec -it glsid2025 bash -c "echo 'SET LINESIZE 120' >> /opt/oracle/product/21c/dbhomeXE/sqlplus/admin/glogin.sql"
docker exec -it glsid2025 bash -c "echo 'COLUMN NomJ FORMAT A20' >> /opt/oracle/product/21c/dbhomeXE/sqlplus/admin/glogin.sql"
```


![alt text](<Capture d’écran du 2025-04-06 18-03-00.png>)

## si le repertoire /home/oracle n'existe pas créer le d'abord


```bash
# Créer d'abord le répertoire home pour l'utilisateur oracle 
docker exec -it glsid2025 mkdir -p /home/oracle
docker exec -it glsid2025 chown oracle:oinstall /home/oracle
```

![alt text](<Capture d’écran du 2025-04-06 18-09-28.png>)

## et si vous rencontrez des problèmes de permissions 

### Solution alternative pour le fichier login.sql

# Utiliser le répertoire /tmp (toujours accessible en écriture) :


```bash
docker exec -it glsid2025 mkdir -p /tmp/oracle_config
```
![alt text](<Capture d’écran du 2025-04-06 18-09-50.png>)

```bash
docker exec -it glsid2025 bash -c "echo '-- Paramètres pour le TP GLSID' > /tmp/oracle_config/login.sql"
docker exec -it glsid2025 bash -c "echo 'SET PAGESIZE 50' >> /tmp/oracle_config/login.sql"
docker exec -it glsid2025 bash -c "echo 'SET LINESIZE 120' >> /tmp/oracle_config/login.sql"
docker exec -it glsid2025 bash -c "echo 'COLUMN NomJ FORMAT A20' >> /tmp/oracle_config/login.sql"
docker exec -it glsid2025 bash -c "echo 'COLUMN PrenomJ FORMAT A20' >> /tmp/oracle_config/login.sql"
```

![alt text](<Capture d’écran du 2025-04-06 18-10-10.png>)

## Vérifier la création du fichier :

```bash
docker exec -it glsid2025 ls -la /tmp/oracle_config/
```

![alt text](<Capture d’écran du 2025-04-06 18-10-37.png>)

## Utiliser le fichier avec SQL*Plus :

```bash
docker exec -it glsid2025 sqlplus -L sys/Glsid2024-2025@XE as sysdba @/tmp/oracle_config/login.sql
```

![alt text](<Capture d’écran du 2025-04-06 18-11-02.png>)

## Premières commandes de vérification (dans SQL*Plus) :

```bash
-- Vérifier les paramètres
SHOW PAGESIZE
SHOW LINESIZE

-- Vérifier la connexion
SELECT * FROM v$version;
SELECT username FROM dba_users;
```

![alt text](<Capture d’écran du 2025-04-06 18-14-18.png>)

![alt text](<Capture d’écran du 2025-04-06 18-15-17.png>)

![alt text](<Capture d’écran du 2025-04-06 18-15-30.png>)

## Création des tables Equpies et Joueurs


```bash
-- Table Equipes
CREATE TABLE Equipes (
    NumEquipe INT NOT NULL,
    Pays VARCHAR2(20),
    CONSTRAINT Equipes_pk PRIMARY KEY (NumEquipe)
);

-- Table Joueurs
CREATE TABLE Joueurs (
    NumJoueur INT NOT NULL,
    NomJ VARCHAR2(20),
    PrenomJ VARCHAR2(20),
    DateNaissance DATE,
    NumEquipe INT,
    CONSTRAINT Joueurs_pk PRIMARY KEY (NumJoueur),
    CONSTRAINT Joueurs_Equipes_fk FOREIGN KEY (NumEquipe) REFERENCES Equipes(NumEquipe)
);
```

![alt text](<Capture d’écran du 2025-04-06 18-16-11.png>)

## Insertion des données 

```bash
-- Equipes
INSERT INTO Equipes VALUES (1, 'Maroc');
INSERT INTO Equipes VALUES (2, 'France');
INSERT INTO Equipes VALUES (3, 'Espagne');
INSERT INTO Equipes VALUES (4, 'Argentine');

-- Joueurs
INSERT INTO Joueurs VALUES (1, 'Ayoubi', 'Rachid', TO_DATE('10/02/1980', 'DD/MM/YYYY'), 1);
INSERT INTO Joueurs VALUES (2, 'Jack', 'Robert', TO_DATE('22/06/1977', 'DD/MM/YYYY'), 2);
INSERT INTO Joueurs VALUES (3, 'Arabi', 'Omar', TO_DATE('16/12/1979', 'DD/MM/YYYY'), 1);
INSERT INTO Joueurs VALUES (4, 'Garcia', 'David', TO_DATE('13/01/1980', 'DD/MM/YYYY'), 3);
INSERT INTO Joueurs VALUES (5, 'Iose', 'George', TO_DATE('30/05/1970', 'DD/MM/YYYY'), 4);
INSERT INTO Joueurs VALUES (6, 'Roberto', 'Claude', TO_DATE('16/10/1981', 'DD/MM/YYYY'), 2);

COMMIT;
```

![alt text](<Capture d’écran du 2025-04-06 18-17-14.png>)

![alt text](<Capture d’écran du 2025-04-06 18-17-28.png>)

![alt text](<Capture d’écran du 2025-04-06 18-17-48.png>)

### Exécution des requêtes 

```bash
-- Requête 1: Noms des joueurs
SELECT NomJ FROM Joueurs;

-- Requête 2: Noms et prénoms
SELECT NomJ, PrenomJ FROM Joueurs;

-- Requête 5: Joueurs du Maroc ou de France
SELECT J.NomJ
FROM Joueurs J, Equipes E
WHERE J.NumEquipe = E.NumEquipe AND (E.Pays='Maroc' OR E.Pays='France');
```

![alt text](<Capture d’écran du 2025-04-06 18-19-01.png>)

![alt text](<Capture d’écran du 2025-04-06 18-19-18.png>)

![alt text](<Capture d’écran du 2025-04-06 18-21-57.png>)

#### Gestion des résultats

## Pour exporter en CSV :

```bash
SPOOL /tmp/oracle_config/resultats.csv
SET MARKUP CSV ON DELIMITER ',' QUOTE ON
SELECT * FROM Joueurs;
SPOOL OFF
```

![alt text](<Capture d’écran du 2025-04-06 18-22-54.png>)

## Récupérer les fichiers :

```bash
docker cp glsid2025:/tmp/oracle_config/resultats.csv .
```

![alt text](<Capture d’écran du 2025-04-06 18-24-44.png>)



####  Requêtes avancées 

```sql

-- Requête 6: Num-Equipe de "Garcia"
SELECT E.NumEquipe 
FROM Joueurs J, Equipes E
WHERE J.NumEquipe = E.NumEquipe AND J.NomJ='Garcia';

-- Requête 7: Nom-Pays de "Jack"
SELECT E.Pays
FROM Joueurs J, Equipes E
WHERE J.NumEquipe = E.NumEquipe AND J.NomJ='Jack';

-- Requête 8: Pays pour noms commençant par "J"
SELECT DISTINCT E.Pays
FROM Joueurs J, Equipes E
WHERE J.NumEquipe = E.NumEquipe AND J.NomJ LIKE 'J%';

-- Requête 9: Pays pour noms finissant par "i"
SELECT DISTINCT E.Pays
FROM Joueurs J, Equipes E
WHERE J.NumEquipe = E.NumEquipe AND J.NomJ LIKE '%i';

```
![alt text](<Capture d’écran du 2025-04-06 18-36-51.png>)

####  Export des résultats

```sql
-- Export texte
SPOOL /tmp/oracle_config/resultat_requete10.txt
SELECT J.NumJoueur
FROM Joueurs J, Equipes E
WHERE J.NumEquipe = E.NumEquipe AND E.Pays='Maroc';
SPOOL OFF

-- Export HTML
SPOOL /tmp/oracle_config/resultat_requete10.html
SET MARKUP HTML ON
SELECT J.NumJoueur
FROM Joueurs J, Equipes E
WHERE J.NumEquipe = E.NumEquipe AND E.Pays='Maroc';
SPOOL OFF
SET MARKUP HTML OFF

-- Export CSV
SPOOL /tmp/oracle_config/resultat_requete10.csv
SET MARKUP CSV ON DELIMITER ',' QUOTE ON
SELECT J.NumJoueur
FROM Joueurs J, Equipes E
WHERE J.NumEquipe = E.NumEquipe AND E.Pays='Maroc';
SPOOL OFF
```
![alt text](image.png)

#### Agrégations et statistiques

```sql
-- 7. Nombre d'équipes par ID
SELECT COUNT(*), NumEquipe
FROM Equipes
GROUP BY NumEquipe;

-- 8. Nombre de joueurs distincts par équipe
SELECT COUNT(DISTINCT NumEquipe)
FROM Joueurs;

-- 9. Equipes avec plus d'un joueur
SELECT COUNT(*), NumEquipe
FROM Joueurs
GROUP BY NumEquipe
HAVING COUNT(*) > 1;
```
![alt text](<Capture d’écran du 2025-04-06 18-51-36.png>)

#### Transaction avec savepoint

```sql
-- Début de transaction
SET TRANSACTION NAME 'TP2_Transaction';
SAVEPOINT avant_modification;

-- Modification test
UPDATE Equipes SET Pays='MOROCCO' WHERE NumEquipe=1;
SELECT * FROM Equipes WHERE NumEquipe=1;

-- Annulation
ROLLBACK TO SAVEPOINT avant_modification;
SELECT * FROM Equipes WHERE NumEquipe=1;

-- Validation
COMMIT COMMENT 'Transaction TP2';
```
![alt text](<Capture d’écran du 2025-04-06 18-52-52.png>)
![alt text](<Capture d’écran du 2025-04-06 18-54-06.png>)

#### Gestion des vues

```sql
-- Création de la vue
CREATE VIEW Vue_Joueurs_Maroc_France AS
SELECT J.NomJ
FROM Joueurs J, Equipes E
WHERE J.NumEquipe = E.NumEquipe AND (E.Pays='Maroc' OR E.Pays='France');

-- Création utilisateur pour la vue
ALTER SESSION SET "_ORACLE_SCRIPT"=TRUE;
CREATE USER TP2vue IDENTIFIED BY TP2vue;
GRANT CONNECT TO TP2vue;
GRANT SELECT ON Vue_Joueurs_Maroc_France TO TP2vue;

-- Test de la vue
CONNECT TP2vue/TP2vue
SELECT * FROM SYS.Vue_Joueurs_Maroc_France;
```
![alt text](<Capture d’écran du 2025-04-06 18-55-18.png>)

#### Gestion des index

Oracle crée automatiquement des index pour les clés primaires.
Il est redondant de créer des index sur des colonnes déjà indexées.
L'intérêt est de créer des index sur :
Des colonnes fréquemment utilisées dans les WHERE
Des colonnes utilisées pour les jointures
Des combinaisons de colonnes (index composites)

```sql
-- Vérification des index
-- Création des index
SELECT index_name, table_name, column_name 
FROM all_ind_columns 
WHERE table_name IN ('EQUIPES', 'JOUEURS')
ORDER BY index_name;

-- Créez des index sur des colonnes non encore indexées

-- Index sur Pays dans Equipes
CREATE INDEX idx_equipes_pays ON SYS.Equipes(Pays);

-- Index sur NomJ dans Joueurs
CREATE INDEX idx_joueurs_nom ON SYS.Joueurs(NomJ);

-- Index composite
CREATE INDEX idx_joueurs_equipe_nom ON SYS.Joueurs(NumEquipe, NomJ);

```
![alt text](<Capture d’écran du 2025-04-06 19-06-00.png>)

#### Nettoyage final

```sql
-- Suppression des enregistrements
DELETE FROM Joueurs;
DELETE FROM Equipes;

-- Suppression des tables
DROP TABLE Joueurs CASCADE CONSTRAINTS;
DROP TABLE Equipes CASCADE CONSTRAINTS;

-- Suppression de la vue
DROP VIEW Vue_Joueurs_Maroc_France;

-- Suppression des utilisateurs
DROP USER TP2 CASCADE;
DROP USER TP2vue CASCADE;
```
 #### Pour récupérer vos fichiers résultats

```bash
docker cp glsid2025:/tmp/oracle_config/resultat_requete10.txt .
docker cp glsid2025:/tmp/oracle_config/resultat_requete10.html .
docker cp glsid2025:/tmp/oracle_config/resultat_requete10.csv .
```
![alt text](<Capture d’écran du 2025-04-06 19-12-22.png>)
![alt text](<Capture d’écran du 2025-04-06 21-30-44.png>)


### Astuces pratiques

## Pour réutiliser facilement votre configuration :

```bash
alias sqlplus-tp='docker exec -it glsid2025 sqlplus -L sys/Glsid2024-2025@XE as sysdba @/tmp/oracle_config/login.sql'
```
## Pour voir les tables créées :

```bash
SELECT table_name FROM user_tables;
```
## Pour décrire une table :

```bash
DESC Joueurs;
```

