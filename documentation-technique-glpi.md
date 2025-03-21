# Documentation technique : Installation d'un serveur GLPI avec l'agent FusionInventory

## Table des matières
1. Introduction
2. Prérequis du système
3. Installation du serveur GLPI sur Debian
4. Configuration de GLPI
5. Installation du plugin FusionInventory
6. Installation de l'agent FusionInventory
7. Configuration de l'agent
8. Vérification du fonctionnement
9. Dépannage
10. Maintenance et sauvegardes
11. Références

## 1. Introduction

Cette documentation technique détaille l'installation d'un serveur GLPI (Gestionnaire Libre de Parc Informatique) avec l'agent FusionInventory pour assurer l'inventaire automatique des équipements. GLPI est une solution open source de gestion des services informatiques (ITSM) permettant la gestion de l'inventaire, le suivi des tickets et la gestion des contrats.

L'intégration de FusionInventory permet d'automatiser la collecte d'informations sur les postes clients et les équipements réseau, offrant ainsi une vue complète et à jour du parc informatique.

## 2. Prérequis du système

### Configuration matérielle recommandée
- Processeur : 2 cœurs minimum (4 cœurs recommandés pour plus de 100 postes)
- Mémoire vive : 4 Go minimum (8 Go recommandés pour plus de 100 postes)
- Espace disque : 20 Go minimum (50 Go recommandés pour la production)
- Connexion réseau : 100 Mbps minimum (1 Gbps recommandé)

### Environnement logiciel
- Système d'exploitation : Debian 11 (Bullseye) ou supérieur
- Serveur web : Apache 2.4 ou plus récent
- Base de données : MariaDB 10.5 ou supérieure
- PHP : Version 8.0 ou supérieure (8.1 recommandé)

### Modules PHP requis
- curl
- gd
- json
- mbstring
- mysqli
- session
- zlib
- simplexml
- xml
- intl
- fileinfo
- apcu
- ldap (recommandé)
- cas (recommandé)
- zip
- bz2
- exif
- opcache

## 3. Installation du serveur GLPI sur Debian

### Préparation du système

Commencez par mettre à jour votre système :

```bash
sudo apt update && sudo apt upgrade -y
```

### Installation des dépendances

Installez les paquets nécessaires :

```bash
sudo apt install -y apache2 mariadb-server php php-curl php-gd php-json php-mbstring php-mysql php-xml php-intl php-zip php-bz2 php-apcu php-ldap php-cas php-fileinfo php-exif libapache2-mod-php php-opcache
```

### Configuration du serveur web Apache

Activez les modules Apache nécessaires :

```bash
sudo a2enmod rewrite
sudo systemctl restart apache2
```

### Configuration de PHP

Créez un fichier de configuration optimisé pour GLPI :

```bash
sudo bash -c 'cat > /etc/php/8.1/apache2/conf.d/99-glpi.ini << EOL
memory_limit = 256M
file_uploads = on
max_execution_time = 600
session.auto_start = 0
session.use_trans_sid = 0
upload_max_filesize = 100M
post_max_size = 100M
date.timezone = Europe/Paris
EOL'

# Redémarrez Apache pour appliquer les changements
sudo systemctl restart apache2
```

### Configuration de MariaDB

Sécurisez l'installation de MariaDB :

```bash
sudo mysql_secure_installation
```

Répondez aux questions suivantes :
- Entrez le mot de passe root actuel (par défaut, appuyez sur Entrée)
- Définissez un nouveau mot de passe root
- Supprimez les utilisateurs anonymes (Y)
- Désactivez la connexion root à distance (Y)
- Supprimez la base de données de test (Y)
- Rechargez les privilèges (Y)

Créez une base de données et un utilisateur pour GLPI :

```bash
sudo mysql -u root -p
```

Dans la console MariaDB, exécutez :

```sql
CREATE DATABASE glpidb CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
CREATE USER 'glpiuser'@'localhost' IDENTIFIED BY 'MotDePasseSecurise';
GRANT ALL PRIVILEGES ON glpidb.* TO 'glpiuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

Optimisez la configuration de MariaDB :

```bash
sudo bash -c 'cat > /etc/mysql/mariadb.conf.d/99-glpi.cnf << EOL
[mysqld]
innodb_buffer_pool_size = 1G
max_allowed_packet = 64M
character-set-server = utf8mb4
collation-server = utf8mb4_general_ci
EOL'

sudo systemctl restart mariadb
```

### Téléchargement et installation de GLPI

Téléchargez la dernière version stable de GLPI :

```bash
VERSION="10.0.10"
wget https://github.com/glpi-project/glpi/releases/download/$VERSION/glpi-$VERSION.tgz
```

Extrayez l'archive et configurez les permissions :

```bash
sudo tar -xzf glpi-$VERSION.tgz -C /var/www/html/
sudo chown -R www-data:www-data /var/www/html/glpi/
sudo find /var/www/html/glpi -type d -exec chmod 755 {} \;
sudo find /var/www/html/glpi -type f -exec chmod 644 {} \;
```

### Configuration du virtualhost Apache

Créez un fichier de configuration pour le virtualhost GLPI :

```bash
sudo bash -c 'cat > /etc/apache2/sites-available/glpi.conf << EOL
<VirtualHost *:80>
    ServerName glpi.example.com
    DocumentRoot /var/www/html/glpi
    
    <Directory /var/www/html/glpi>
        Options FollowSymLinks
        AllowOverride All
        Require all granted
        
        # Protection des fichiers sensibles
        <FilesMatch ".(sql|yml|conf)$">
            Require all denied
        </FilesMatch>
    </Directory>
    
    # Protection du répertoire d'installation
    <Directory /var/www/html/glpi/install>
        <IfModule mod_authz_core.c>
            Require local
        </IfModule>
    </Directory>
    
    ErrorLog ${APACHE_LOG_DIR}/glpi_error.log
    CustomLog ${APACHE_LOG_DIR}/glpi_access.log combined
</VirtualHost>
EOL'
```

Activez le site et redémarrez Apache :

```bash
sudo a2ensite glpi.conf
sudo systemctl restart apache2
```

### Configuration des tâches planifiées

Configurez le cron pour GLPI :

```bash
sudo bash -c 'cat > /etc/cron.d/glpi << EOL
# GLPI core cron tasks
*/15 * * * * www-data /usr/bin/php /var/www/html/glpi/front/cron.php
EOL'
```

## 4. Configuration de GLPI

Accédez à l'interface web de GLPI via votre navigateur : `http://adresse-serveur/glpi`

### Assistant d'installation

1. Sélectionnez la langue d'installation.
2. Acceptez les termes de la licence.
3. Cliquez sur "Installer".
4. Sélectionnez "Nouvelle installation".
5. Vérifiez les prérequis système (tous les indicateurs doivent être verts).
6. Entrez les informations de connexion à la base de données :
   - Hôte : localhost
   - Utilisateur : glpiuser
   - Mot de passe : MotDePasseSecurise
   - Base de données : glpidb
7. Sélectionnez la base de données.
8. Initialisez la base de données (patientez pendant cette étape).
9. Créez et configurez le compte administrateur avec un mot de passe fort.
10. Configurez les paramètres généraux selon vos besoins.

### Première connexion et sécurisation

1. Une fois l'installation terminée, connectez-vous avec le compte que vous avez créé.
2. Si vous n'avez pas créé de compte administrateur personnalisé, utilisez ces identifiants par défaut :
   - Identifiant : glpi
   - Mot de passe : glpi
3. Changez immédiatement le mot de passe par défaut.
4. Configurez les paramètres de sécurité dans "Configuration" > "Générale" > "Sécurité".
5. Supprimez le répertoire d'installation pour des raisons de sécurité :
   ```bash
   sudo rm -rf /var/www/html/glpi/install/
   ```

## 5. Installation du plugin FusionInventory

### Téléchargement et installation du plugin

```bash
# Téléchargement du plugin FusionInventory
cd /var/www/html/glpi/plugins/
sudo wget https://github.com/fusioninventory/fusioninventory-for-glpi/releases/download/glpi10.0.10-1.1/fusioninventory-10.0.10-1.1.tar.gz
sudo tar -xzf fusioninventory-10.0.10-1.1.tar.gz
sudo rm fusioninventory-10.0.10-1.1.tar.gz
sudo chown -R www-data:www-data fusioninventory/
```

### Activation du plugin dans l'interface GLPI

1. Connectez-vous à l'interface d'administration de GLPI.
2. Allez dans "Configuration" > "Plugins".
3. Localisez "FusionInventory" dans la liste des plugins.
4. Cliquez sur "Installer".
5. Une fois installé, cliquez sur "Activer".

### Configuration du plugin FusionInventory

1. Allez dans "Configuration" > "FusionInventory" > "Configuration générale".
2. Configurez les paramètres selon vos besoins :
   - Fréquence des inventaires
   - Entités autorisées
   - Options de découverte
   - Comportement à l'égard des équipements inconnus

3. Configurez les tâches planifiées pour FusionInventory :
   ```bash
   sudo bash -c 'cat >> /etc/cron.d/glpi << EOL
   # FusionInventory cron tasks
   */15 * * * * www-data /usr/bin/php /var/www/html/glpi/plugins/fusioninventory/front/cron.php
   EOL'
   ```

## 6. Installation de l'agent FusionInventory

### Installation de l'agent sur Debian

```bash
# Installation du dépôt
sudo apt-get install -y lsb-release wget gnupg
sudo bash -c "echo 'deb https://debian.fusioninventory.org/downloads `lsb_release -cs` main' > /etc/apt/sources.list.d/fusioninventory-agent.list"
wget -O - https://debian.fusioninventory.org/debian/archive.key | sudo apt-key add -
sudo apt-get update

# Installation de l'agent
sudo apt-get install -y fusioninventory-agent
```

### Installation de l'agent sur Windows

1. Téléchargez l'installateur Windows depuis le [site officiel de FusionInventory](https://fusioninventory.org/documentation/agent/installation/windows.html).
2. Exécutez l'installateur avec les droits administrateur.
3. Lors de l'installation, configurez l'URL du serveur : `http://adresse-serveur/glpi/plugins/fusioninventory/`.
4. Configurez la fréquence d'exécution (par défaut : 24 heures).
5. Sélectionnez les modules à activer selon vos besoins.

### Installation de l'agent sur MacOS

1. Téléchargez le package DMG depuis le [site officiel de FusionInventory](https://fusioninventory.org/documentation/agent/installation/macos.html).
2. Ouvrez le package et suivez les instructions d'installation.
3. Configurez l'agent via le terminal :
   ```bash
   sudo defaults write /Library/Preferences/org.fusioninventory.agent server http://adresse-serveur/glpi/plugins/fusioninventory/
   ```

## 7. Configuration de l'agent

### Configuration de l'agent sur Debian

Éditez le fichier de configuration principal :

```bash
sudo nano /etc/fusioninventory/agent.cfg
```

Configurez les paramètres suivants :

```
server = http://adresse-serveur/glpi/plugins/fusioninventory/
local = /var/lib/fusioninventory-agent
logfile = /var/log/fusioninventory-agent/fusioninventory.log
timeout = 180
delaytime = 60
no-category = environment
no-httpd = 0
tag = MonTag
```

Redémarrez l'agent :

```bash
sudo systemctl restart fusioninventory-agent
```

### Configuration spécifique par système

#### Paramètres importants à configurer

- **Server** : URL du serveur GLPI avec le chemin vers le plugin FusionInventory
- **Tag** : Étiquette permettant de regrouper les machines (optionnel)
- **Delaytime** : Temps aléatoire d'attente avant l'exécution (pour éviter l'engorgement du serveur)
- **Scan-homedirs** : Activer/désactiver le scan des répertoires personnels
- **No-category** : Catégories à exclure de l'inventaire

### Déploiement en masse

Pour un déploiement en masse sur un parc informatique, vous pouvez :

1. Créer des packages préconfigurés avec des outils comme WAPT ou PDQ Deploy.
2. Utiliser des GPO pour Windows pour déployer l'agent et ses configurations.
3. Utiliser des outils comme Ansible pour les environnements Linux.
4. Exemple de script Ansible pour déployer l'agent sur Debian :
   ```yaml
   - name: Install FusionInventory Agent
     hosts: all
     become: yes
     tasks:
       - name: Add FusionInventory repository
         shell: echo 'deb https://debian.fusioninventory.org/downloads `lsb_release -cs` main' > /etc/apt/sources.list.d/fusioninventory-agent.list
         
       - name: Add repository key
         shell: wget -O - https://debian.fusioninventory.org/debian/archive.key | apt-key add -
         
       - name: Update apt cache
         apt:
           update_cache: yes
           
       - name: Install FusionInventory agent
         apt:
           name: fusioninventory-agent
           state: present
           
       - name: Configure agent
         template:
           src: agent.cfg.j2
           dest: /etc/fusioninventory/agent.cfg
           
       - name: Restart agent
         service:
           name: fusioninventory-agent
           state: restarted
           enabled: yes
   ```

## 8. Vérification du fonctionnement

### Lancement manuel d'un inventaire

Pour vérifier que l'agent fonctionne correctement, lancez manuellement un inventaire :

Sur Debian :
```bash
sudo fusioninventory-agent --force
```

Sur Windows :
```
"C:\Program Files\FusionInventory-Agent\fusioninventory-agent.exe" --force
```

Sur MacOS :
```bash
sudo /usr/local/bin/fusioninventory-agent --force
```

### Vérification dans GLPI

1. Connectez-vous à l'interface GLPI.
2. Allez dans "Parc" > "Ordinateurs".
3. Vérifiez que le nouvel ordinateur est présent dans la liste.
4. Consultez les détails de l'ordinateur pour confirmer que l'inventaire est complet.

### Vérification des journaux

Consultez les journaux pour détecter d'éventuels problèmes :

Sur le serveur :
```bash
sudo tail -f /var/log/apache2/error.log
```

Sur l'agent Debian :
```bash
sudo tail -f /var/log/fusioninventory-agent/fusioninventory.log
```

## 9. Dépannage

### Problèmes courants et solutions

| Problème | Cause probable | Solution |
|----------|----------------|----------|
| L'agent ne se connecte pas au serveur | URL incorrecte ou pare-feu | Vérifiez l'URL du serveur dans la configuration de l'agent et assurez-vous que les pare-feu autorisent la connexion |
| Erreurs dans les journaux de l'agent | Problèmes de configuration | Activez le mode debug (`--debug`) pour plus d'informations et vérifiez la configuration |
| Inventaire incomplet | Restrictions d'accès ou configuration | Vérifiez les paramètres "no-category" et les droits d'accès de l'agent |
| Base de données indisponible | Service MariaDB arrêté | Vérifiez l'état du service MariaDB avec `systemctl status mariadb` |
| Erreurs PHP | Modules manquants ou configuration inadéquate | Vérifiez que tous les modules PHP requis sont installés et correctement configurés |

### Commandes de diagnostic

```bash
# Vérifier la configuration de l'agent
fusioninventory-agent --config

# Tester la connexion au serveur
curl -I http://adresse-serveur/glpi/plugins/fusioninventory/

# Vérifier le statut des services
systemctl status apache2 mariadb fusioninventory-agent

# Vérifier les journaux Apache
tail -f /var/log/apache2/error.log

# Vérifier les journaux MariaDB
tail -f /var/log/mysql/error.log

# Vérifier les permissions des fichiers GLPI
ls -la /var/www/html/glpi/
```

## 10. Maintenance et sauvegardes

### Sauvegardes régulières

Créez un script de sauvegarde pour GLPI :

```bash
#!/bin/bash
# Script de sauvegarde pour GLPI
DATE=$(date +%Y%m%d)
BACKUP_DIR="/var/backups/glpi"

# Création du répertoire de sauvegarde
mkdir -p $BACKUP_DIR

# Sauvegarde de la base de données
mysqldump -u glpiuser -p'MotDePasseSecurise' glpidb | gzip > $BACKUP_DIR/glpidb_$DATE.sql.gz

# Sauvegarde des fichiers
tar -czf $BACKUP_DIR/glpi_files_$DATE.tar.gz /var/www/html/glpi/files/

# Suppression des sauvegardes de plus de 30 jours
find $BACKUP_DIR -name "*.gz" -type f -mtime +30 -delete
```

Programmez l'exécution de ce script avec cron :

```bash
sudo bash -c 'cat > /etc/cron.d/glpi-backup << EOL
# Sauvegarde quotidienne de GLPI à 2h du matin
0 2 * * * root /chemin/vers/script/backup-glpi.sh
EOL'
```

### Mises à jour de sécurité

Configurez les mises à jour automatiques pour le système :

```bash
sudo apt install -y unattended-upgrades
sudo dpkg-reconfigure unattended-upgrades
```

### Surveillance du système

Mettez en place une solution de surveillance comme Nagios ou Zabbix pour surveiller :
- L'état des services (Apache, MariaDB)
- L'espace disque disponible
- L'utilisation de la mémoire et du processeur
- La connectivité réseau

## 11. Références

- [Documentation officielle GLPI](https://glpi-project.org/documentation/)
- [Documentation officielle FusionInventory](https://fusioninventory.org/documentation/)
- [Forum GLPI](https://forum.glpi-project.org/)
- [Forum FusionInventory](https://forum.fusioninventory.org/)
- [Wiki GLPI](https://wiki.glpi-project.org/)
- [Dépôt GitHub de GLPI](https://github.com/glpi-project/glpi)
- [Dépôt GitHub de FusionInventory](https://github.com/fusioninventory/fusioninventory-for-glpi)
