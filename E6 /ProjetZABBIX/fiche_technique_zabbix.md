## 1. Présentation du projet

### 1.1 Objectifs de la supervision

Ce projet vise à mettre en place une solution complète de supervision réseau utilisant **Zabbix 7.0.13**. Les objectifs principaux sont :

- Surveiller en temps réel l'état des serveurs et services de l'infrastructure
- Collecter et analyser les métriques système (CPU, RAM, espace disque, réseau)
- Détecter proactivement les incidents et dysfonctionnements
- Fournir un tableau de bord centralisé pour le monitoring
- Établir une base pour l'extension future de la supervision

### 1.2 Contexte pédagogique (BTS SIO – SISR)

Cette réalisation s'inscrit dans le cadre de la formation **BTS Services Informatiques aux Organisations**, option **Solutions d'Infrastructure, Systèmes et Réseaux (SISR)**. Le projet permet de mettre en pratique les compétences suivantes :

- Administration de systèmes Linux (Ubuntu Server)
- Administration de systèmes Windows Server
- Configuration de services réseau
- Mise en place d'outils de supervision
- Gestion de la sécurité réseau (pare-feu, ports)

### 1.3 Environnement de test (machines virtuelles sur VirtualBox)

L'infrastructure est entièrement virtualisée sur **VirtualBox**, permettant un environnement de test sécurisé et facilement reproductible. Cette approche offre plusieurs avantages :

- Isolation complète de l'environnement de production
- Possibilité de créer des snapshots pour revenir en arrière
- Facilité de déploiement et de reconfiguration
- Coût réduit (pas de matériel physique dédié)

---

## 2. Architecture réseau

### 2.1 Description du réseau 192.168.114.0/24

L'infrastructure repose sur un réseau local privé de classe C avec les caractéristiques suivantes :

- **Réseau** : 192.168.114.0/24
- **Masque de sous-réseau** : 255.255.255.0
- **Plage d'adresses disponibles** : 192.168.114.1 à 192.168.114.254
- **Type** : Réseau interne VirtualBox

### 2.2 Topologie réseau

L'architecture repose sur deux machines virtuelles interconnectées dans un réseau privé, avec des flux de communication bidirectionnels pour la supervision. Le serveur Zabbix initie les connexions vers l'agent pour collecter les métriques, tandis que l'agent peut également transmettre des données de manière proactive selon la configuration.

### 2.3 Adressage IP, rôle de chaque machine

| Machine | Adresse IP | Rôle principal | Services |
|---------|------------|----------------|----------|
| **SRV-ZAB-GUA** | 192.168.114.50 | Serveur de supervision | Zabbix Server, Apache2, PHP 8.3, MariaDB |
| **SRV-DC-GUA** | 192.168.114.88 | Serveur supervisé | Windows Server, DC, DNS, DHCP, Zabbix Agent 2 |

**Ports utilisés :**
- **10051** : Communication Zabbix Server (écoute)
- **10050** : Communication Zabbix Agent (écoute)
- **80** : Interface web Zabbix Frontend

---

## 3. Serveur Zabbix (SRV-ZAB-GUA)

### 3.1 Système : Ubuntu Server 22.04

Le serveur de supervision fonctionne sur **Ubuntu Server 22.04 LTS**, choisi pour plusieurs raisons techniques :

**Avantages d'Ubuntu Server 22.04 LTS :**
- Support à long terme jusqu'en 2027 (Long Term Support)
- Compatibilité optimale avec Zabbix 7.0.13
- Gestion native des paquets via APT
- Sécurité renforcée avec mises à jour régulières
- Performance optimisée pour les services réseau
- Documentation extensive et communauté active

**Configuration système de base :**
- Installation minimale sans interface graphique
- Configuration réseau statique pour éviter les conflits DHCP
- Mise à jour complète du système avant installation des services
- Configuration du fuseau horaire Europe/Paris
- Activation du service SSH pour l'administration distante

### 3.2 Rôles installés : Zabbix Server, Apache2, PHP, MariaDB

L'architecture du serveur repose sur une pile LAMP adaptée pour Zabbix :

- **Zabbix Server 7.0.13** : Moteur principal de supervision
- **Apache2** : Serveur web pour l'interface graphique
- **PHP 8.3** : Langage de script pour le frontend web
- **MariaDB** : Base de données pour stocker les configurations et données de supervision

### 3.3 Installation des paquets Zabbix 7.0.13

L'installation suit la procédure officielle Zabbix avec ajout du dépôt de packages :

**Étapes détaillées d'installation :**

```bash
# 1. Téléchargement et installation du dépôt Zabbix
wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_7.0-2+ubuntu22.04_all.deb
sudo dpkg -i zabbix-release_7.0-2+ubuntu22.04_all.deb

# 2. Mise à jour de la liste des paquets
sudo apt update

# 3. Installation des composants Zabbix
sudo apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent2 -y

# 4. Installation des dépendances système
sudo apt install apache2 php php-mysql mariadb-server mariadb-client -y
```

**Paquets installés et leurs rôles :**
- **zabbix-server-mysql** : Serveur principal Zabbix avec support MySQL/MariaDB
- **zabbix-frontend-php** : Interface web en PHP pour la gestion
- **zabbix-apache-conf** : Configuration Apache pré-définie pour Zabbix
- **zabbix-sql-scripts** : Scripts de création de base de données
- **zabbix-agent2** : Agent local pour auto-supervision du serveur

**Vérification post-installation :**
```bash
# Vérification des services installés
systemctl status zabbix-server
systemctl status apache2
systemctl status mariadb

# Vérification des ports d'écoute
netstat -tulpn | grep :10051
netstat -tulpn | grep :80
```

### 3.4 Configuration de la base de données

La base de données MariaDB nécessite une configuration complète et sécurisée :

**Étapes de configuration MariaDB :**

```bash
# 1. Sécurisation initiale de MariaDB
sudo mysql_secure_installation
# - Définition du mot de passe root
# - Suppression des utilisateurs anonymes
# - Désactivation de l'accès root distant
# - Suppression de la base de test

# 2. Connexion à MariaDB et création de la base
sudo mysql -u root -p
```

```sql
-- Création de la base de données Zabbix
CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;

-- Création de l'utilisateur Zabbix
CREATE USER 'zabbix'@'localhost' IDENTIFIED BY 'mot_de_passe_securise';

-- Attribution des privilèges
GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'localhost';

-- Application des changements
FLUSH PRIVILEGES;

-- Vérification de la création
SHOW DATABASES;
SELECT User, Host FROM mysql.user WHERE User = 'zabbix';
```

**Import des données initiales :**
```bash
# Localisation du fichier de schéma
find /usr/share/zabbix-sql-scripts/ -name "*.sql.gz"

# Import du schéma de base de données
zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql -u zabbix -p zabbix

# Vérification de l'import
mysql -u zabbix -p zabbix -e "SHOW TABLES;"
```

**Optimisation de MariaDB pour Zabbix :**

Modification du fichier `/etc/mysql/mariadb.conf.d/50-server.cnf` :
```ini
[mysqld]
innodb_buffer_pool_size = 128M
innodb_file_per_table = 1
innodb_flush_log_at_trx_commit = 2
innodb_flush_method = O_DIRECT
query_cache_size = 32M
query_cache_type = 1
max_connections = 200
```

**Configuration du serveur Zabbix :**

Modification du fichier `/etc/zabbix/zabbix_server.conf` :
```ini
# Configuration base de données
DBName=zabbix
DBUser=zabbix
DBPassword=mot_de_passe_securise
DBHost=localhost
DBPort=3306

# Configuration système
LogFile=/var/log/zabbix/zabbix_server.log
LogFileSize=10
PidFile=/var/run/zabbix/zabbix_server.pid

# Configuration réseau
ListenPort=10051
ListenIP=0.0.0.0

# Configuration des processus
StartPollers=10
StartIPMIPollers=5
StartPollersUnreachable=3
StartTrappers=5
StartPingers=3
StartDiscoverers=3
StartHTTPPollers=3

# Timeout et cache
Timeout=30
CacheSize=32M
HistoryCacheSize=16M
HistoryIndexCacheSize=4M
TrendCacheSize=4M
ValueCacheSize=8M
```

### 3.5 Configuration Apache et PHP

**Configuration détaillée de PHP** (`/etc/php/8.3/apache2/php.ini`) :

```ini
# Optimisation des ressources
max_execution_time = 300
max_input_time = 300
memory_limit = 128M
post_max_size = 16M
upload_max_filesize = 2M

# Configuration des sessions
session.auto_start = 0
session.cookie_httponly = 1
session.use_only_cookies = 1

# Paramètres Zabbix spécifiques
date.timezone = Europe/Paris
always_populate_raw_post_data = -1

# Extensions requises (à vérifier)
extension=mysqli
extension=gettext
extension=mbstring
extension=sockets
extension=bcmath
extension=gd
extension=xml
extension=ctype
extension=session
extension=xmlreader
extension=xmlwriter
```

**Vérification des extensions PHP requises :**
```bash
# Vérification des modules PHP installés
php -m | grep -E "(mysqli|gettext|mbstring|sockets|bcmath|gd|xml|ctype|session)"

# Installation des extensions manquantes si nécessaire
sudo apt install php8.3-mysql php8.3-gettext php8.3-mbstring php8.3-sockets php8.3-bcmath php8.3-gd php8.3-xml -y
```

**Configuration Apache pour Zabbix :**

Le fichier `/etc/apache2/conf-available/zabbix.conf` est automatiquement configuré par le paquet `zabbix-apache-conf` :

```apache
# Configuration Zabbix automatique
Alias /zabbix /usr/share/zabbix

<Directory "/usr/share/zabbix">
    Options FollowSymLinks
    AllowOverride None
    Order allow,deny
    Allow from all
    
    # Configuration PHP pour Zabbix
    php_value max_execution_time 300
    php_value memory_limit 128M
    php_value post_max_size 16M
    php_value upload_max_filesize 2M
    php_value max_input_time 300
    php_value date.timezone Europe/Paris
</Directory>

<Directory "/usr/share/zabbix/conf">
    Order deny,allow
    Deny from all
    <files "*.conf">
        Order deny,allow
        Deny from all
    </files>
</Directory>
```

**Activation des modules et redémarrage :**
```bash
# Activation de la configuration Zabbix
sudo a2enconf zabbix

# Activation des modules Apache nécessaires
sudo a2enmod rewrite
sudo a2enmod ssl

# Redémarrage des services
sudo systemctl restart apache2
sudo systemctl restart php8.3-fpm
```

**Configuration de la sécurité réseau :**

```bash
# Activation et configuration d'UFW (Uncomplicated Firewall)
sudo ufw enable

# Autorisation SSH (port 22)
sudo ufw allow ssh

# Autorisation HTTP pour l'interface web Zabbix
sudo ufw allow 80/tcp

# Autorisation du port Zabbix Server
sudo ufw allow 10051/tcp

# Vérification des règles appliquées
sudo ufw status verbose

# Test d'écoute des ports
sudo netstat -tulpn | grep -E ":80|:10051"
```

**Démarrage et activation des services :**
```bash
# Démarrage des services essentiels
sudo systemctl start zabbix-server
sudo systemctl start apache2
sudo systemctl start mariadb

# Activation au démarrage
sudo systemctl enable zabbix-server
sudo systemctl enable apache2
sudo systemctl enable mariadb

# Vérification du statut
sudo systemctl status zabbix-server
sudo systemctl status apache2
sudo systemctl status mariadb
```

**Test de fonctionnement :**
- **Interface web** : `http://192.168.114.50/zabbix`
- **Vérification des logs** : `sudo tail -f /var/log/zabbix/zabbix_server.log`
- **Test de connectivité** : `telnet 192.168.114.50 10051`

**Configuration initiale du frontend web :**

Lors du premier accès à l'interface web, assistant de configuration automatique :
1. Vérification des prérequis PHP
2. Configuration de la connexion à la base de données
3. Paramètres du serveur Zabbix
4. Résumé de la configuration
5. Création du compte administrateur par défaut (Admin/zabbix)

---

## 4. Agent supervisé (SRV-DC-GUA)

### 4.1 Système : Windows Server

Le serveur supervisé fonctionne sous **Windows Server 2019/2022**, configuré comme environnement de production simulé représentant un serveur d'infrastructure critique.

**Spécifications techniques :**
- **Système d'exploitation** : Windows Server 2019 Standard ou 2022
- **RAM allouée** : 4 GB minimum pour les services DC/DNS/DHCP
- **Stockage** : 60 GB d'espace disque virtuel
- **Processeur** : 2 vCPU minimum
- **Carte réseau** : Adaptateur réseau interne VirtualBox

**Rôles Windows Server installés :**
- **Active Directory Domain Services (AD DS)** : Service d'annuaire central
- **DNS Server** : Résolution de noms pour le domaine
- **DHCP Server** : Attribution automatique d'adresses IP
- **Remote Desktop Services** : Accès à distance pour l'administration

### 4.2 Rôle : Contrôleur de domaine + serveur DHCP + agent Zabbix

**Services Windows installés :**
- **Active Directory Domain Services** : Contrôleur de domaine
- **DNS Server** : Serveur de noms de domaine
- **DHCP Server** : Attribution automatique d'adresses IP
- **Zabbix Agent 2** : Agent de supervision

**Configuration DNS :**
- Serveur DNS principal pour le domaine local
- Redirecteurs DNS configurés (8.8.8.8) pour l'accès Internet
- Résolution DNS fonctionnelle depuis le serveur

### 4.3 Fixation de l'IP statique

**Configuration réseau détaillée :**

**Paramètres TCP/IP :**
- **Adresse IP** : 192.168.114.88
- **Masque de sous-réseau** : 255.255.255.0 (/24)
- **Passerelle par défaut** : 192.168.114.1
- **DNS primaire** : 192.168.114.88 (auto-référence pour AD)
- **DNS secondaire** : 8.8.8.8 (Google DNS pour Internet)

**Procédure de configuration via PowerShell :**
```powershell
# Configuration de l'interface réseau
$InterfaceIndex = (Get-NetAdapter).InterfaceIndex
New-NetIPAddress -InterfaceIndex $InterfaceIndex -IPAddress 192.168.114.88 -PrefixLength 24 -DefaultGateway 192.168.114.1

# Configuration des serveurs DNS
Set-DnsClientServerAddress -InterfaceIndex $InterfaceIndex -ServerAddresses "192.168.114.88","8.8.8.8"

# Vérification de la configuration
Get-NetIPConfiguration
Get-DnsClientServerAddress
```

**Configuration via l'interface graphique :**
1. Panneau de configuration → Centre Réseau et partage
2. Modifier les paramètres de la carte réseau
3. Propriétés → Protocole Internet Version 4 (TCP/IPv4)
4. Utiliser l'adresse IP suivante
5. Saisie des paramètres réseau
6. Validation et test de connectivité

**Tests de validation réseau :**
```cmd
# Test de connectivité locale
ping 192.168.114.88

# Test de connectivité vers le serveur Zabbix
ping 192.168.114.50

# Test de résolution DNS
nslookup google.com
nslookup srv-zab-gua.domaine.local

# Test de connectivité Internet
ping 8.8.8.8
```

**Configuration DNS avancée :**
- **Zone de recherche directe** : domaine.local
- **Zone de recherche inversée** : 114.168.192.in-addr.arpa
- **Redirecteurs DNS** : 8.8.8.8 et 8.8.4.4 configurés pour les requêtes externes
- **Enregistrements DNS** : Enregistrements A et PTR pour les serveurs de l'infrastructure

### 4.4 Installation de Zabbix Agent (ou Agent 2)

**Choix de Zabbix Agent 2 :**

Zabbix Agent 2 a été sélectionné pour ses avantages techniques :
- **Performance améliorée** : Écrit en Go, plus rapide que l'agent classique
- **Compatibilité native Windows** : Meilleure intégration système
- **Plugins intégrés** : Support natif de nombreux services
- **Configuration simplifiée** : Syntaxe de configuration plus intuitive
- **Monitoring étendu** : Capacités de surveillance avancées

**Procédure d'installation détaillée :**

**1. Téléchargement :**
```powershell
# Téléchargement depuis PowerShell
$url = "https://www.zabbix.com/downloads/7.0.13/zabbix_agent2-7.0.13-windows-amd64-openssl.msi"
$output = "$env:TEMP\zabbix_agent2.msi"
Invoke-WebRequest -Uri $url -OutFile $output
```

**2. Installation silencieuse via MSI :**
```cmd
# Installation en mode silencieux avec paramètres pré-configurés
msiexec /i zabbix_agent2.msi /quiet ^
SERVER=192.168.114.50 ^
SERVERACTIVE=192.168.114.50 ^
HOSTNAME=SRV-DC-GUA ^
INSTALLFOLDER="C:\Program Files\Zabbix Agent 2"
```

**3. Installation manuelle via l'interface graphique :**
- Exécution du fichier MSI téléchargé
- Assistant d'installation avec paramètres personnalisés
- Configuration des paramètres de base pendant l'installation
- Création automatique du service Windows

**4. Vérification post-installation :**
```powershell
# Vérification du service
Get-Service -Name "Zabbix Agent 2"

# Vérification des fichiers installés
Get-ChildItem "C:\Program Files\Zabbix Agent 2"

# Test de connectivité réseau
Test-NetConnection -ComputerName 192.168.114.50 -Port 10051
```

### 4.5 Configuration du fichier `zabbix_agentd.conf`

**Localisation du fichier de configuration :**
- **Chemin principal** : `C:\Program Files\Zabbix Agent 2\zabbix_agentd.conf`
- **Fichier de sauvegarde** : Créer une copie avant modification

**Configuration complète du fichier :**

```ini
############ GENERAL PARAMETERS #################

# Liste des serveurs Zabbix autorisés à se connecter
Server=192.168.114.50

# Serveur Zabbix pour les vérifications actives
ServerActive=192.168.114.50

# Nom d'hôte de l'agent (doit correspondre exactement à celui configuré sur le serveur)
Hostname=SRV-DC-GUA

############ ADVANCED PARAMETERS #################

# Port d'écoute de l'agent
ListenPort=10050

# Interface d'écoute (0.0.0.0 pour toutes les interfaces)
ListenIP=0.0.0.0

# Adresse source pour les connexions sortantes
SourceIP=192.168.114.88

############ LOGGING PARAMETERS ##################

# Fichier de log
LogFile=C:\Program Files\Zabbix Agent 2\zabbix_agentd.log

# Taille maximale du fichier de log (Mo)
LogFileSize=10

# Niveau de log (0=aucun, 1=critique, 2=erreur, 3=warning, 4=info, 5=debug)
DebugLevel=3

############ SECURITY PARAMETERS #################

# Timeout pour les opérations réseau
Timeout=10

# Répertoire pour les fichiers PID
PidFile=C:\Program Files\Zabbix Agent 2\zabbix_agentd.pid

############ PERFORMANCE PARAMETERS ##############

# Buffer pour les métriques (nombre d'éléments)
BufferSize=100

# Nombre maximum de nouvelles connexions par seconde
MaxLinesPerSecond=20

############ USER-DEFINED MONITORED PARAMETERS ###

# Paramètres personnalisés (exemples)
# UserParameter=custom.disk.discovery,powershell -NoProfile -Command "Get-WmiObject Win32_LogicalDisk | ConvertTo-Json"
# UserParameter=custom.service[*],powershell -NoProfile -Command "Get-Service '$1' | Select-Object Status | ConvertTo-Json"

############ LOADABLE MODULES #####################

# Modules supplémentaires (optionnel)
# LoadModulePath=C:\Program Files\Zabbix Agent 2\modules
# LoadModule=zabbix_module_dummy.dll

############ TLS PARAMETERS #######################

# Configuration TLS (optionnel pour sécurisation)
# TLSConnect=cert
# TLSAccept=cert
# TLSCAFile=C:\Program Files\Zabbix Agent 2\certs\ca.crt
# TLSCertFile=C:\Program Files\Zabbix Agent 2\certs\agent.crt
# TLSKeyFile=C:\Program Files\Zabbix Agent 2\certs\agent.key
```

**Paramètres critiques à vérifier :**

1. **Hostname** : Doit correspondre exactement au nom configuré dans l'interface Zabbix
2. **Server/ServerActive** : Adresse IP du serveur Zabbix (192.168.114.50)
3. **ListenPort** : Port d'écoute (10050 par défaut)
4. **DebugLevel** : Niveau 3 recommandé pour le debugging initial

**Application de la configuration :**

```powershell
# Redémarrage du service pour appliquer les modifications
Restart-Service "Zabbix Agent 2"

# Vérification du statut du service
Get-Service "Zabbix Agent 2" | Format-Table Name, Status, StartType

# Vérification des logs
Get-Content "C:\Program Files\Zabbix Agent 2\zabbix_agentd.log" -Tail 20
```

**Tests de validation de la configuration :**

```cmd
# Test de connectivité depuis le serveur Zabbix (à exécuter sur SRV-ZAB-GUA)
zabbix_get -s 192.168.114.88 -k agent.ping
zabbix_get -s 192.168.114.88 -k system.cpu.load[all,avg1]
zabbix_get -s 192.168.114.88 -k vm.memory.size[available]

# Test de résolution DNS inverse
nslookup 192.168.114.88
```

### 4.6 Ajout de l'exception pare-feu pour le port 10050

**Configuration détaillée du pare-feu Windows :**

**Méthode 1 : Via PowerShell (recommandée) :**

```powershell
# Création de la règle d'entrée pour Zabbix Agent
New-NetFirewallRule -DisplayName "Zabbix Agent 2" -Direction Inbound -Protocol TCP -LocalPort 10050 -Action Allow -Profile Any

# Création d'une règle plus restrictive (optionnel)
New-NetFirewallRule -DisplayName "Zabbix Agent 2 Secure" -Direction Inbound -Protocol TCP -LocalPort 10050 -RemoteAddress 192.168.114.50 -Action Allow -Profile Any

# Vérification des règles créées
Get-NetFirewallRule -DisplayName "*Zabbix*" | Format-Table DisplayName, Direction, Action, Enabled

# Test de connectivité
Test-NetConnection -ComputerName 192.168.114.88 -Port 10050
```

**Méthode 2 : Via l'interface graphique Windows :**

1. **Accès au pare-feu :**
   - Panneau de configuration → Système et sécurité → Pare-feu Windows Defender
   - Paramètres avancés

2. **Création de la règle d'entrée :**
   - Clic droit sur "Règles de trafic entrant" → Nouvelle règle
   - Type de règle : Port
   - Protocole : TCP
   - Port local spécifique : 10050
   - Action : Autoriser la connexion
   - Profils : Domaine, Privé, Public (tous cochés)
   - Nom : "Zabbix Agent 2 - Port 10050"
   - Description : "Autorisation du trafic entrant pour l'agent Zabbix depuis le serveur de supervision"

3. **Configuration avancée de sécurité (optionnel) :**
   - Propriétés de la règle → Onglet "Étendue"
   - Adresses IP distantes : Adresses spécifiques
   - Ajout de l'IP : 192.168.114.50
   - Cette configuration limite l'accès au seul serveur Zabbix

**Méthode 3 : Via la ligne de commande (netsh) :**

```cmd
# Ajout de la règle pare-feu
netsh advfirewall firewall add rule name="Zabbix Agent 2" dir=in action=allow protocol=TCP localport=10050

# Vérification de la règle
netsh advfirewall firewall show rule name="Zabbix Agent 2"

# Règle plus restrictive avec IP source
netsh advfirewall firewall add rule name="Zabbix Agent 2 Secure" dir=in action=allow protocol=TCP localport=10050 remoteip=192.168.114.50
```

**Tests de connectivité et validation :**

**1. Test depuis le serveur Windows :**
```powershell
# Vérification de l'écoute sur le port 10050
netstat -an | findstr :10050

# Test de connectivité locale
Test-NetConnection -ComputerName localhost -Port 10050
Test-NetConnection -ComputerName 192.168.114.88 -Port 10050

# Vérification des logs de l'agent
Get-Content "C:\Program Files\Zabbix Agent 2\zabbix_agentd.log" -Tail 10
```

**2. Test depuis le serveur Zabbix (SRV-ZAB-GUA) :**
```bash
# Test de connectivité réseau basique
telnet 192.168.114.88 10050

# Test avec zabbix_get (plus précis)
zabbix_get -s 192.168.114.88 -k agent.ping
zabbix_get -s 192.168.114.88 -k agent.version
zabbix_get -s 192.168.114.88 -k system.hostname

# Test de métriques système
zabbix_get -s 192.168.114.88 -k system.cpu.util[,idle]
zabbix_get -s 192.168.114.88 -k vm.memory.size[available]
zabbix_get -s 192.168.114.88 -k vfs.fs.size[C:,free]
```

## 5. Configuration dans Zabbix Web

### 5.1 Accès à l'interface d'administration

**URL d'accès** : `http://192.168.114.50/zabbix`

**Identifiants par défaut** :
- Utilisateur : `Admin`
- Mot de passe : `zabbix`

### 5.2 Création manuelle de l'hôte

#### Étapes de création :

1. **Navigation** : Configuration → Hosts → Create host
2. **Paramètres de l'hôte** :
   ```
   Host name: SRV-DC-GUA
   Visible name: SRV-DC-GUA (Windows Server)
   Groups: Windows servers
   ```

3. **Configuration de l'interface** :
   ```
   Interface type: Agent
   IP address: 192.168.114.88
   DNS name: (laisser vide si résolution par IP)
   Connect to: IP
   Port: 10050
   ```

### 5.3 Ajout de l'interface agent et application du template OS Windows

#### Configuration de l'interface agent :

**Paramètres techniques** :
- **Type** : Zabbix agent
- **Adresse IP** : 192.168.114.88
- **Port** : 10050 (port par défaut de Zabbix Agent)
- **Connexion** : IP (recommandé pour éviter les problèmes DNS)

#### Application du template :

1. **Sélection du template** : `Template OS Windows by Zabbix agent`
2. **Ajout** : Onglet "Templates" → Link new templates
3. **Validation** : Cliquer sur "Add" puis "Update"

**Métriques supervisées par le template** :
- Utilisation CPU (%)
- Consommation mémoire RAM
- Espace disque disponible
- Processus système
- Services Windows critiques
- Performances réseau

### 5.4 Réglages du nom d'hôte

#### Correspondance obligatoire :

Le paramètre `Hostname` dans la configuration de l'agent Zabbix **doit strictement correspondre** au nom d'hôte déclaré dans l'interface web.

**Fichier de configuration agent** (`C:\Program Files\Zabbix Agent 2\zabbix_agent2.conf`) :
```ini
Hostname=SRV-DC-GUA
```

**Interface web Zabbix** :
```
Host name: SRV-DC-GUA
```

⚠️ **Attention** : Toute incohérence entre ces deux valeurs provoque un état "ZBX gris" (agent inaccessible).

---

## 6. Tests et vérifications

### 6.1 Ping entre les machines

#### Test de connectivité réseau :

**Depuis SRV-ZAB-GUA vers SRV-DC-GUA** :
```bash
ping 192.168.114.88
```

**Depuis SRV-DC-GUA vers SRV-ZAB-GUA** :
```cmd
ping 192.168.114.50
```

**Résultat attendu** : Réponse sans perte de paquets (0% packet loss)

### 6.2 Test `zabbix_get` depuis le serveur

#### Commande de test depuis SRV-ZAB-GUA :

```bash
zabbix_get -s 192.168.114.88 -p 10050 -k agent.ping
```

**Résultats possibles** :
- `1` : Agent accessible et fonctionnel
- `Connection refused` : Port 10050 fermé ou service arrêté
- `Timeout` : Problème réseau ou pare-feu

#### Tests additionnels :

**Test de version de l'agent** :
```bash
zabbix_get -s 192.168.114.88 -p 10050 -k agent.version
```

**Test d'utilisation CPU** :
```bash
zabbix_get -s 192.168.114.88 -p 10050 -k system.cpu.util[,avg1]
```

### 6.3 État ZBX dans le dashboard

#### Vérification visuelle :

1. **Navigation** : Monitoring → Hosts
2. **Indicateur ZBX** : 
   - 🟢 **Vert** : Agent accessible
   - 🔴 **Rouge** : Agent inaccessible
   - ⚪ **Gris** : Pas de données ou configuration incorrecte

#### Temps de mise à jour :

- **Délai normal** : 30 secondes à 2 minutes
- **Forcer l'actualisation** : Configuration → Hosts → sélectionner l'hôte → "Check now"

### 6.4 Vérification des services et ports actifs

#### Côté Windows (SRV-DC-GUA) :

**Vérification du service Zabbix** :
```cmd
sc query "Zabbix Agent 2"
```

**Vérification du port d'écoute** :
```cmd
netstat -an | findstr :10050
```

**Résultat attendu** :
```
TCP    0.0.0.0:10050         0.0.0.0:0              LISTENING
```

#### Côté serveur Zabbix (SRV-ZAB-GUA) :

**Vérification du service Zabbix Server** :
```bash
sudo systemctl status zabbix-server
```

**Vérification du port d'écoute** :
```bash
sudo netstat -tlnp | grep :10051
```

### 6.5 Résolution du problème "agent indisponible"

#### Checklist de diagnostic :

1. **Connectivité réseau** :
   ```bash
   telnet 192.168.114.88 10050
   ```

2. **Service agent** :
   ```cmd
   net start "Zabbix Agent 2"
   ```

3. **Pare-feu Windows** :
   - Vérifier la règle pour le port 10050
   - Test avec pare-feu temporairement désactivé

4. **Configuration hostname** :
   - Correspondance exacte entre agent et interface web
   - Redémarrage du service après modification

---

## 7. Incidents rencontrés et résolus

### 7.1 Problème de hostname incohérent

#### Symptôme :
- État ZBX gris dans le dashboard
- Message d'erreur : "received value for unknown host"

#### Cause :
Différence entre le `Hostname` configuré dans l'agent et celui déclaré dans l'interface web.

#### Résolution :
1. **Vérification côté agent** :
   ```ini
   # Dans zabbix_agent2.conf
   Hostname=SRV-DC-GUA
   ```

2. **Vérification côté web** :
   - Configuration → Hosts → Modifier l'hôte
   - S'assurer que "Host name" = "SRV-DC-GUA"

3. **Redémarrage du service** :
   ```cmd
   net stop "Zabbix Agent 2"
   net start "Zabbix Agent 2"
   ```

### 7.2 Port 10050 bloqué

#### Symptôme :
- `zabbix_get` retourne "Connection refused"
- Telnet vers le port 10050 échoue

#### Cause :
Pare-feu Windows bloque le port entrant 10050.

#### Résolution :
1. **Créer une règle de pare-feu** :
   ```cmd
   netsh advfirewall firewall add rule name="Zabbix Agent" dir=in action=allow protocol=TCP localport=10050
   ```

2. **Ou via l'interface graphique** :
   - Pare-feu Windows Defender → Paramètres avancés
   - Règles de trafic entrant → Nouvelle règle
   - Port TCP 10050 → Autoriser la connexion

### 7.3 Carte réseau sur mauvais réseau NAT

#### Symptôme :
- Machines ne se pinguent pas
- Adresses IP sur des sous-réseaux différents

#### Cause :
VirtualBox a assigné les machines sur des réseaux NAT différents.

#### Résolution :
1. **Arrêter les VMs**
2. **Configuration réseau** :
   - Paramètres VM → Réseau
   - Adaptateur 1 : Réseau interne
   - Nom : "LAN_Zabbix" (même nom pour les deux VMs)

3. **Redémarrer et vérifier la connectivité**

### 7.4 Renommage de machine = perte d'accès

#### Symptôme :
Après renommage de la machine Windows, l'agent devient inaccessible.

#### Cause :
Le hostname dans la configuration de l'agent ne correspond plus au nom système.

#### Résolution :
1. **Mettre à jour la configuration** :
   ```ini
   Hostname=NOUVEAU-NOM-MACHINE
   ```

2. **Modifier l'hôte dans Zabbix Web** :
   - Host name : NOUVEAU-NOM-MACHINE

3. **Redémarrer le service agent**

### 7.5 Blocage DNS côté DC sans redirecteurs

#### Symptôme :
- Le contrôleur de domaine ne résout plus les noms externes
- Perte de connectivité Internet

#### Cause :
DNS configuré en tant que serveur autoritaire sans redirecteurs.

#### Résolution :
1. **Ouvrir DNS Manager**
2. **Clic droit sur le serveur → Propriétés**
3. **Onglet Redirecteurs → Ajouter** :
   - 8.8.8.8 (Google DNS)
   - 8.8.4.4 (Google DNS secondaire)

4. **Redémarrer le service DNS**

---

## 8. Sécurisation et optimisation

### 8.1 Fixation des IPs

#### Configuration IP statique SRV-ZAB-GUA :
```bash
# /etc/netplan/00-installer-config.yaml
network:
  ethernets:
    enp0s3:
      addresses:
        - 192.168.114.50/24
      gateway4: 192.168.114.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```

#### Configuration IP statique SRV-DC-GUA :
- Interface réseau → Propriétés IPv4
- IP : 192.168.114.88
- Masque : 255.255.255.0
- Passerelle : 192.168.114.1
- DNS : 127.0.0.1, 8.8.8.8

### 8.2 Redémarrage automatique des services

#### Service Zabbix Server (Ubuntu) :
```bash
sudo systemctl enable zabbix-server
sudo systemctl enable zabbix-agent
sudo systemctl enable apache2
sudo systemctl enable mariadb
```

#### Service Zabbix Agent (Windows) :
```cmd
sc config "Zabbix Agent 2" start= auto
sc config "Zabbix Agent 2" failure reset= 60 actions= restart/30000/restart/30000/restart/30000
```

### 8.3 Libération mémoire sur le serveur

#### Optimisation MariaDB :
```ini
# /etc/mysql/mariadb.conf.d/50-server.cnf
[mysqld]
innodb_buffer_pool_size = 256M
query_cache_size = 32M
query_cache_limit = 2M
```

#### Optimisation Zabbix Server :
```ini
# /etc/zabbix/zabbix_server.conf
StartPollers=5
StartTrappers=5
StartPingers=1
StartDiscoverers=1
CacheSize=32M
HistoryCacheSize=16M
TrendCacheSize=8M
```

### 8.4 Conseils sur l'usage de l'agent 2

#### Avantages de Zabbix Agent 2 :
- **Performance** : Écrit en Go, plus rapide que l'agent 1
- **Plugins** : Architecture modulaire extensible
- **Ressources** : Consommation mémoire réduite
- **Compatibilité** : Supporte les items de l'agent 1

#### Configuration recommandée :
```ini
Server=192.168.114.50
ServerActive=192.168.114.50
Hostname=SRV-DC-GUA
RefreshActiveChecks=60
BufferSend=5
BufferSize=100
MaxLinesPerSecond=20
Timeout=3
```

---

## 9. Évolutions possibles

### 9.1 Supervision d'autres machines (Linux, Windows)

#### Déploiement à grande échelle :

**Templates recommandés** :
- `Template OS Linux by Zabbix agent` : Serveurs Linux
- `Template OS Windows by Zabbix agent` : Postes/serveurs Windows
- `Template Net Cisco IOS by SNMP` : Équipements réseau Cisco
- `Template DB MySQL by Zabbix agent` : Bases de données MySQL

**Stratégie de déploiement** :
1. **Standardisation des configurations** : Scripts d'installation automatisés
2. **Groupement logique** : Organisation par fonction (web, bdd, file servers)
3. **Templates personnalisés** : Adaptation aux besoins métier

### 9.2 Intégration d'un mail d'alerte

#### Configuration du media type email :

1. **Administration → Media types → Email**
2. **Paramètres SMTP** :
   ```
   SMTP server: smtp.gmail.com
   SMTP server port: 587
   SMTP helo: zabbix.local
   SMTP email: supervision@entreprise.com
   Security: STARTTLS
   Authentication: Username and password
   ```

#### Création d'actions automatisées :

**Conditions de déclenchement** :
- Seuil CPU > 85% pendant 5 minutes
- Espace disque < 10%
- Service critique arrêté
- Perte de connectivité réseau

### 9.3 Règles de découverte automatique

#### Auto-découverte réseau :

**Configuration Discovery** :
```
IP range: 192.168.114.1-254
Update interval: 1h
Checks: Zabbix agent (port 10050)
Device uniqueness: IP address
Host name: IP address
```

**Actions automatiques** :
- Ajout automatique des hôtes détectés
- Application de templates par OS détecté
- Notification des nouveaux équipements

### 9.4 Sauvegarde des métriques

#### Stratégie de rétention :

**Configuration dans Zabbix** :
```
History storage period: 90 days
Trend storage period: 5 years
```

**Sauvegarde base de données** :
```bash
#!/bin/bash
# Script de sauvegarde quotidien
mysqldump -u zabbix -p zabbix > /backup/zabbix_$(date +%Y%m%d).sql
find /backup -name "zabbix_*.sql" -mtime +30 -delete
```

### 9.5 Surveillance de services personnalisés

#### Monitoring applicatif avancé :

**Services Windows personnalisés** :
```ini
# Surveillance d'un service spécifique
UserParameter=service.custom[*],powershell -Command "Get-Service -Name $1 | Select-Object -ExpandProperty Status"
```

**Applications web** :
- Monitoring de disponibilité HTTP/HTTPS
- Temps de réponse des pages
- Codes de statut et contenu de réponse
- Certificats SSL et dates d'expiration

**Bases de données** :
- Connexions actives
- Requêtes lentes
- Taille des tables et index
- Réplication et synchronisation

---
