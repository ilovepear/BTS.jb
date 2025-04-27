# BTS Services Informatiques aux Organisations
Option Solutions d'Infrastructure, Systèmes et Réseaux
Session 2025
E6 : Parcours de Professionnalisation :
Projet Personnalisé Encadré 3 (PPE 3) :
PPE 5 : Mainenance du VPN à Oodrive : Cybersécurité

# Table des matières
Nature de l'activité : .............................................................................................................................................. 5
Contexte : .......................................................................................................................................................... 5
Objectifs : .......................................................................................................................................................... 5
Conditions de réalisations : ................................................................................................................................. 5
Matériel : ............................................................................................................................................................ 5
Contraintes : ...................................................................................................................................................... 5
Logiciel : ............................................................................................................................................................ 5
Requis : ............................................................................................................................................................. 5
Difficultés rencontrées : ................................................................................................................................... 5
Durée de réalisation : ...................................................................................................................................... 5
Solutions envisageables : ................................................................................................................................... 5
Solution retenue : ................................................................................................................................................. 5
Conditions initiales : ......................................................................................................................................... 5
Résultat final : ................................................................................................................................................... 5
Outils utilisés : .................................................................................................................................................. 5
Compétences mise en œuvre pour cette activité professionnelle : .............................................................. 5
I. Introduction : ................................................................................................................................................. 6
II. Présentation de l'infrastructure VPN : ........................................................................................................ 6
a. Architecture du réseau VPN : ............................................................................................................... 6
b. Configuration des instances VPN : ...................................................................................................... 7
c. Sécurisation des accès : ....................................................................................................................... 8
d. Certificats et authentification : ............................................................................................................. 10
e. Journalisation et monitoring : ............................................................................................................. 11
III. Procédures de maintenance : ............................................................................................................... 12
a. Mise à jour du serveur VPN : .............................................................................................................. 12
b. Gestion des certificats utilisateurs : ................................................................................................... 14
IV. Gestion des accès utilisateurs : ........................................................................................................... 15
a. Création de compte utilisateur : ......................................................................................................... 15
b. Révocation d'accès : ........................................................................................................................... 16
c. Modification des privilèges d'accès : ................................................................................................. 16
V. Surveillance et résolution d'incidents : .................................................................................................... 17
a. Outils de monitoring : .......................................................................................................................... 17
b. Procédures d'intervention : ................................................................................................................ 18
c. Plan de reprise d'activité : .................................................................................................................. 19
VI. Documentation et conformité : ............................................................................................................ 20
a. Mise à jour de la documentation : ..................................................................................................... 20
b. Audits de sécurité : ............................................................................................................................. 20
VII. Conclusion : .......................................................................................................................................... 21

# Nature de l'activité :
## Contexte :
Dans le cadre de mon activité au sein d'Oodrive, entreprise certifiée SecNumCloud, je dois assurer la maintenance de l'infrastructure VPN interne permettant l'accès sécurisé aux ressources de l'entreprise pour les collaborateurs en télétravail et dans les différents bureaux. Cette infrastructure est critique car elle constitue le point d'entrée au réseau d'entreprise et doit répondre aux exigences strictes de la certification SecNumCloud.

## Objectifs :
Assurer le bon fonctionnement et la sécurité des trois instances VPN (normale, développement et production), gérer les accès utilisateurs, effectuer les mises à jour de sécurité et maintenir la conformité avec les standards de sécurité requis par la certification SecNumCloud.

## Conditions de réalisations :
### Matériel :
- Serveurs dédiés pour chaque instance VPN (3 serveurs)
- Matériel réseau Cisco pour la sécurisation des flux
- HSM (Hardware Security Module) pour la sécurisation des clés privées
- Station d'administration sécurisée pour la gestion des serveurs VPN

### Contraintes :
- Conformité aux exigences SecNumCloud
- Haute disponibilité (99,9%)
- Gestion stricte des accès par profils utilisateurs
- Traçabilité complète des actions et connexions
- Mise à jour régulière sans interruption de service

### Logiciel :
- OpenVPN pour l'infrastructure VPN
- PuTTY pour l'accès SSH aux serveurs
- EasyRSA pour la gestion des certificats
- Zabbix pour le monitoring
- Elastic Stack pour la centralisation des logs

### Requis :
- Accès administrateur aux serveurs VPN
- Clés privées sécurisées pour la génération des certificats
- Documentation technique des infrastructures
- Procédures de sécurité Oodrive

### Difficultés rencontrées :
- Gestion des révocations de certificats lors des départs de collaborateurs
- Maintien de la haute disponibilité pendant les mises à jour
- Conformité continue avec les évolutions des exigences SecNumCloud

### Durée de réalisation :
Activité continue avec interventions planifiées mensuelles (4 heures) et maintenance corrective ponctuelle.

## Solutions envisageables :
- Solution commerciale VPN avec support externalisé
- Infrastructure VPN open-source gérée en interne
- Solution hybride avec VPN managé et personnalisations internes

## Solution retenue :
Infrastructure OpenVPN gérée en interne avec trois instances distinctes (normale, développement, production) pour répondre aux besoins spécifiques d'accès aux ressources et garantir une séparation stricte des environnements.

### Conditions initiales :
Infrastructure VPN fonctionnelle mais nécessitant une maintenance régulière pour assurer sa conformité continue et son adaptation aux besoins évolutifs de l'entreprise.

### Résultat final :
Infrastructure VPN hautement disponible, sécurisée, conforme aux standards SecNumCloud et permettant un accès différencié aux ressources selon les profils utilisateurs.

### Outils utilisés :
OpenVPN, PuTTY, EasyRSA, OpenSSL, Zabbix, Elastic Stack, scripts bash personnalisés pour l'automatisation.

## Compétences mise en œuvre pour cette activité professionnelle :
- A.1.1.1 Analyse du cahier des charges d'un service à produire
- A.1.1.3 Étude des exigences liées à la qualité attendue d'un service
- A.1.3.4 Déploiement d'un service
- A.1.4.1 Participation à un projet
- A.2.3.1 Identification, qualification et évaluation d'un problème
- A.2.3.2 Proposition d'amélioration d'un service
- A.3.1.1 Administration d'une infrastructure
- A.3.1.2 Administration des moyens techniques d'un environnement de communications unifiées
- A.3.1.3 Suivi d'incidents et de remédiations
- A.3.2.1 Installation et configuration d'éléments d'infrastructure
- A.3.2.2 Remplacement ou mise à jour d'éléments défectueux ou obsolètes
- A.3.3.1 Administration d'une base de données
- A.3.3.3 Sécurisation d'une solution technique
- A.4.1.2 Recueil des besoins d'évolution d'une solution d'infrastructure
- A.4.1.7 Développement, utilisation ou adaptation de composants logiciels
- A.4.1.9 Rédaction d'une documentation technique
- A.5.1.1 Mise en place d'une gestion de configuration
- A.5.1.2 Recueil d'informations sur une configuration et ses éléments
- A.5.1.3 Suivi d'une configuration et de ses éléments
- A.5.1.4 Étude de propositions de contrats de service (client, fournisseur)
- A.5.1.5 Évaluation d'un élément de configuration ou d'une configuration
- A.5.1.6 Évaluation d'un investissement informatique
- A.5.2.1 Gestion des actifs informatiques
- A.5.2.2 Gestion des problèmes et des changements
- A.5.2.3 Gestion des risques informatiques
- A.5.2.4 Gestion de la qualité de service informatique

# I. Introduction :
Dans le cadre de mes responsabilités au sein de l'équipe IT d'Oodrive, j'assure la maintenance et la gestion de l'infrastructure VPN interne de l'entreprise. Cette infrastructure est particulièrement critique car Oodrive est certifiée SecNumCloud, certification française de haut niveau pour les fournisseurs de services cloud qui impose des exigences strictes en matière de sécurité et de disponibilité des systèmes.

Le VPN interne constitue la porte d'entrée sécurisée au réseau d'entreprise pour les collaborateurs en télétravail, dont le nombre a considérablement augmenté ces dernières années. Il permet également de sécuriser les communications entre les différents sites d'Oodrive et d'assurer un accès contrôlé aux ressources sensibles.

Pour répondre aux besoins spécifiques d'Oodrive en termes de séparation des environnements, l'infrastructure VPN est composée de trois instances distinctes :
- VPN Normal : pour l'accès aux ressources courantes de l'entreprise (bureautique, intranet, applications internes)
- VPN Développement : pour l'accès aux environnements de développement et de test
- VPN Production : pour l'accès aux environnements de production, hautement sécurisé et limité aux équipes opérationnelles

Cette organisation permet de respecter le principe de séparation des environnements imposé par la certification SecNumCloud tout en facilitant la gestion des accès selon les profils utilisateurs.

# II. Présentation de l'infrastructure VPN :
## a. Architecture du réseau VPN :

L'infrastructure VPN d'Oodrive repose sur une architecture distribuée avec trois serveurs dédiés, chacun hébergeant une instance spécifique du VPN. Ces serveurs sont situés dans le datacenter principal d'Oodrive et bénéficient des mesures de sécurité physique et logique associées.

Chaque instance VPN dispose de sa propre plage d'adresses IP privées pour éviter les conflits de routage :
- VPN Normal : 10.10.0.0/16
- VPN Développement : 10.20.0.0/16
- VPN Production : 10.30.0.0/16

Le schéma ci-dessous illustre l'architecture globale du réseau VPN :

![Architecture du réseau VPN](Figure 1 Architecture du réseau VPN)

Les connexions VPN utilisent le protocole OpenVPN en mode TLS avec une double authentification :
- Certificat client (authentification mutuelle TLS)
- Nom d'utilisateur et mot de passe (authentification secondaire)

Cette double authentification répond aux exigences de la certification SecNumCloud en matière d'authentification forte.

Les flux réseau sont contrôlés par des pare-feux dédiés qui filtrent les accès selon les règles définies pour chaque instance VPN :
- Le pare-feu d'entrée filtre les connexions entrantes vers les serveurs VPN
- Les pare-feux internes contrôlent les accès aux ressources de l'entreprise selon le profil de l'utilisateur et l'instance VPN utilisée

## b. Configuration des instances VPN :

Les trois instances VPN sont configurées de manière similaire mais avec des paramètres de sécurité et d'accès spécifiques à leur rôle.

**VPN Normal :**
- Port UDP 1194
- Chiffrement AES-256-GCM
- Compression LZ4
- MTU optimisé à 1400
- Authentification par certificat et credentials
- Accès aux ressources standard de l'entreprise
- Journalisation standard des connexions

Extrait de la configuration du serveur VPN Normal :
```
port 1194
proto udp
dev tun
ca ca.crt
cert server.crt
key server.key
dh dh2048.pem
server 10.10.0.0 255.255.0.0
ifconfig-pool-persist ipp.txt
push "route 172.16.0.0 255.255.0.0"
push "dhcp-option DNS 10.10.0.1"
keepalive 10 120
cipher AES-256-GCM
auth SHA256
compress lz4-v2
push "compress lz4-v2"
user nobody
group nogroup
persist-key
persist-tun
status openvpn-status.log
log-append /var/log/openvpn/openvpn.log
verb 3
```

**VPN Développement :**
- Port UDP 1195
- Chiffrement AES-256-GCM
- Compression LZ4
- MTU optimisé à 1400
- Authentification par certificat et credentials
- Accès aux environnements de développement et de test
- Journalisation avancée des connexions et actions

**VPN Production :**
- Port UDP 1196
- Chiffrement AES-256-GCM avec clé renforcée (renouvelée toutes les 4 heures)
- Sans compression (pour éviter les vulnérabilités liées à la compression)
- MTU optimisé à 1400
- Double authentification renforcée (certificat, credentials et OTP)
- Accès limité aux environnements de production
- Journalisation exhaustive des connexions et actions
- Session limitée à 8 heures maximum

## c. Sécurisation des accès :

La sécurisation des accès aux serveurs VPN est un élément crucial de l'infrastructure, particulièrement dans le contexte de la certification SecNumCloud qui exige un niveau élevé de protection.

**Accès administrateur aux serveurs VPN :**

L'accès aux serveurs VPN pour les opérations de maintenance est strictement contrôlé :
- Connexion SSH uniquement depuis des postes d'administration identifiés
- Authentification par clé SSH avec passphrase (pas d'authentification par mot de passe)
- Restriction des adresses IP sources autorisées
- Utilisation de PuTTY comme client SSH standard
- Journalisation complète des sessions d'administration

La configuration SSH des serveurs VPN est renforcée :

```
# Configuration SSH sécurisée
Port 2222
Protocol 2
HostKey /etc/ssh/ssh_host_ed25519_key
HostKey /etc/ssh/ssh_host_rsa_key

# Algorithmes de chiffrement sécurisés
KexAlgorithms curve25519-sha256@libssh.org,diffie-hellman-group16-sha512,diffie-hellman-group18-sha512
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com

# Authentification
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
PermitEmptyPasswords no
AuthenticationMethods publickey

# Restrictions
AllowUsers admin-vpn
AllowGroups sysadmin
```

**Protection contre les attaques :**

Pour protéger l'infrastructure VPN contre les attaques, plusieurs mécanismes sont mis en place :

- Protection anti-DDoS en amont des serveurs VPN
- Limitation du nombre de tentatives de connexion (fail2ban)
- Analyseur d'intrusion (IDS/IPS) surveillant les flux vers les serveurs VPN
- Vérification de l'intégrité des fichiers système (AIDE)

Configuration de fail2ban pour la protection des serveurs VPN :

```
[openvpn]
enabled = true
port = 1194,1195,1196
protocol = udp
filter = openvpn
logpath = /var/log/openvpn/openvpn.log
maxretry = 3
bantime = 3600

[sshd]
enabled = true
port = 2222
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600
```

## d. Certificats et authentification :

La gestion des certificats est un élément central de la sécurité de l'infrastructure VPN d'Oodrive. Chaque utilisateur dispose d'un certificat personnel pour se connecter aux instances VPN auxquelles il a accès.

**Infrastructure de certificats :**

L'infrastructure de certificats repose sur une PKI (Public Key Infrastructure) interne gérée avec EasyRSA :
- Une autorité de certification (CA) racine offline, stockée de manière sécurisée
- Une autorité de certification intermédiaire pour la signature des certificats utilisateurs
- Des certificats serveur spécifiques pour chaque instance VPN
- Des certificats client individuels pour chaque utilisateur

La clé privée de l'autorité de certification racine est protégée par un HSM (Hardware Security Module) pour garantir sa sécurité.

**Génération de certificats utilisateurs :**

La création d'un nouveau certificat utilisateur suit une procédure stricte :

1. Connexion SSH au serveur de gestion des certificats
2. Génération de la demande de certificat (CSR) avec EasyRSA :

```bash
cd /etc/openvpn/easy-rsa/
source vars
./easyrsa gen-req utilisateur nopass
```

3. Signature du certificat par l'autorité de certification intermédiaire :

```bash
./easyrsa sign-req client utilisateur
```

4. Création du fichier de configuration OpenVPN pour l'utilisateur :

```bash
cat > utilisateur.ovpn << EOF
client
dev tun
proto udp
remote vpn.oodrive.com 1194
resolv-retry infinite
nobind
persist-key
persist-tun
remote-cert-tls server
cipher AES-256-GCM
auth SHA256
key-direction 1
verb 3
<ca>
$(cat /etc/openvpn/ca.crt)
</ca>
<cert>
$(cat /etc/openvpn/issued/utilisateur.crt)
</cert>
<key>
$(cat /etc/openvpn/private/utilisateur.key)
</key>
<tls-auth>
$(cat /etc/openvpn/ta.key)
</tls-auth>
EOF
```

5. Transmission sécurisée du fichier de configuration à l'utilisateur

**Révocation de certificats :**

En cas de départ d'un collaborateur ou de compromission suspectée, les certificats doivent être révoqués immédiatement :

```bash
cd /etc/openvpn/easy-rsa/
source vars
./easyrsa revoke utilisateur
./easyrsa gen-crl
```

La liste de révocation de certificats (CRL) est ensuite déployée sur les serveurs VPN pour bloquer toute tentative de connexion avec ces certificats révoqués.

## e. Journalisation et monitoring :

La journalisation et le monitoring de l'infrastructure VPN sont essentiels pour assurer sa sécurité et sa conformité avec les exigences SecNumCloud.

**Journalisation :**

Chaque instance VPN génère plusieurs types de logs :
- Logs de connexion (qui se connecte, quand, depuis quelle adresse IP)
- Logs d'authentification (succès/échecs)
- Logs de trafic (volumes, destinations)
- Logs système des serveurs VPN

Ces logs sont centralisés dans une infrastructure Elastic Stack (Elasticsearch, Logstash, Kibana) pour faciliter leur analyse et leur archivage sécurisé.

Configuration de la journalisation OpenVPN :

```
status /var/log/openvpn/openvpn-status.log
log-append /var/log/openvpn/openvpn.log
verb 3
status-version 2
```

**Monitoring :**

Le monitoring des serveurs VPN est assuré par Zabbix qui surveille en temps réel :
- L'état des services OpenVPN
- La charge des serveurs (CPU, mémoire, disque)
- Le nombre de connexions actives
- Les tentatives d'authentification échouées
- Le trafic réseau

Des alertes sont configurées pour notifier l'équipe d'administration en cas d'anomalie :
- Service OpenVPN arrêté ou redémarré
- Nombre important d'échecs d'authentification
- Charge serveur anormalement élevée
- Espace disque critique

**Tableau de bord de supervision :**

Un tableau de bord Kibana permet de visualiser en temps réel l'état de l'infrastructure VPN et d'identifier rapidement les anomalies :

![Tableau de bord de supervision VPN](Figure 2 Tableau de bord de supervision VPN)

# III. Procédures de maintenance :
## a. Mise à jour du serveur VPN :

La mise à jour des serveurs VPN est une tâche critique qui doit être réalisée régulièrement pour garantir la sécurité de l'infrastructure, tout en minimisant l'impact sur les utilisateurs.

**Planification des mises à jour :**

Les mises à jour sont planifiées selon leur criticité :
- Mises à jour de sécurité critiques : application immédiate selon une procédure d'urgence
- Mises à jour de sécurité normales : application hebdomadaire pendant une fenêtre de maintenance
- Mises à jour fonctionnelles : application mensuelle après validation en environnement de test

Chaque mise à jour fait l'objet d'une notification préalable aux utilisateurs, sauf en cas d'urgence critique.

**Procédure de mise à jour standard :**

1. Vérification des mises à jour disponibles :

```bash
ssh admin@vpn-server
sudo apt update
apt list --upgradable
```

2. Création d'un snapshot du serveur pour permettre un rollback en cas de problème

3. Application des mises à jour :

```bash
sudo apt upgrade -y
```

4. Vérification spécifique des mises à jour OpenVPN :

```bash
apt-cache policy openvpn
```

5. Redémarrage contrôlé du service OpenVPN :

```bash
sudo systemctl restart openvpn@server
```

6. Vérification du bon fonctionnement après mise à jour :

```bash
sudo systemctl status openvpn@server
tail -f /var/log/openvpn/openvpn.log
```

**Procédure de mise à jour avec impact :**

Lorsqu'une mise à jour nécessite un redémarrage du serveur ou une interruption prolongée du service, une procédure spécifique est appliquée pour minimiser l'impact :

1. Notification aux utilisateurs 72h avant l'intervention

2. Configuration du serveur de secours pour prendre le relais :

```bash
# Sur le serveur de secours
sudo cp /etc/openvpn/server/server.conf.standby /etc/openvpn/server/server.conf
sudo systemctl start openvpn@server
```

3. Mise à jour du DNS pour rediriger les connexions vers le serveur de secours

4. Application des mises à jour sur le serveur principal

5. Tests complets de validation

6. Bascule retour vers le serveur principal

7. Notification de fin d'intervention aux utilisateurs

**Gestion des versions :**

Un tableau de suivi des versions est maintenu pour chaque composant de l'infrastructure VPN :

| Composant | Version actuelle | Dernière mise à jour | Commentaires |
|-----------|------------------|----------------------|--------------|
| OpenVPN | 2.5.7 | 12/03/2025 | CVE-2023-XXXX corrigée |
| OpenSSL | 3.0.7 | 15/02/2025 | |
| Kernel | 5.15.0-91 | 20/03/2025 | |
| EasyRSA | 3.1.0 | 05/01/2025 | |

## b. Gestion des certificats utilisateurs :

La gestion du cycle de vie des certificats utilisateurs est une tâche récurrente dans la maintenance de l'infrastructure VPN.

**Création de certificats :**

La création de nouveaux certificats suit un processus formalisé initié par une demande validée par le responsable hiérarchique :

1. Réception de la demande via le système de ticketing

2. Vérification des droits d'accès selon le profil utilisateur

3. Génération du certificat et du fichier de configuration :

```bash
ssh admin@cert-server
cd /etc/openvpn/easy-rsa/
source vars
./easyrsa gen-req utilisateur nopass
./easyrsa sign-req client utilisateur
```

4. Création du fichier de configuration spécifique pour l'instance VPN concernée :

Pour le VPN normal :
```bash
./generate-config.sh utilisateur normal > utilisateur-normal.ovpn
```

Pour le VPN développement :
```bash
./generate-config.sh utilisateur dev > utilisateur-dev.ovpn
```

Pour le VPN production (si autorisé) :
```bash
./generate-config.sh utilisateur prod > utilisateur-prod.ovpn
```

5. Transmission sécurisée du fichier de configuration à l'utilisateur via un lien de téléchargement à usage unique et limité dans le temps

6. Mise à jour de la base de données des certificats

**Renouvellement de certificats :**

Les certificats utilisateurs ont une validité limitée (1 an) et doivent être renouvelés avant expiration :

1. Identification des certificats arrivant à expiration (script automatique mensuel)

```bash
./check-expiring-certs.sh 30
```

2. Notification aux utilisateurs concernés 30 jours avant expiration

3. Génération des nouveaux certificats :

```bash
./easyrsa renew utilisateur nopass
```

4. Distribution des nouveaux fichiers de configuration

**Révocation de certificats :**

La révocation des certificats est nécessaire lors du départ d'un collaborateur ou en cas de suspicion de compromission :

1. Révocation du certificat :

```bash
./easyrsa revoke utilisateur
```

2. Génération de la nouvelle liste de révocation (CRL) :

```bash
./easyrsa gen-crl
```

3. Déploiement de la CRL sur les serveurs VPN :

```bash
scp crl.pem admin@vpn-normal:/etc/openvpn/server/
scp crl.pem admin@vpn-dev:/etc/openvpn/server/
scp crl.pem admin@vpn-prod:/etc/openvpn/server/
```

4. Mise à jour de la base de données des certificats

5. Notification au service RH et au responsable sécurité

# IV. Gestion des accès utilisateurs :
## a. Création de compte utilisateur :

La création d'un compte utilisateur VPN suit un processus formalisé qui commence par une demande validée par la hiérarchie et se termine par la configuration complète des accès.

**Processus de demande d'accès :**

1. Soumission de la demande via le portail intranet avec les informations suivantes :
   - Identité du demandeur
   - Profil d'accès souhaité (normal, développement, production)
   - Justification professionnelle
   - Validation hiérarchique

2. Validation par l'équipe sécurité qui vérifie la conformité de la demande avec la politique d'accès

3. Création du ticket d'intervention pour l'équipe infrastructure

**Configuration des accès :**

1. Création du compte utilisateur sur le serveur d'authentification :

```bash
ssh admin@auth-server
sudo useradd -m -s /bin/false vpn-utilisateur
sudo passwd vpn-utilisateur
```

2. Attribution des groupes d'accès selon le profil :

```bash
# Pour accès VPN normal
sudo usermod -a -G vpn-users vpn-utilisateur

# Pour accès VPN développement
sudo usermod -a -G vpn-dev vpn-utilisateur

# Pour accès VPN production (si autorisé)
sudo usermod -a -G vpn-prod vpn-utilisateur
```

3. Génération du certificat client comme décrit précédemment

4. Configuration des règles de filtrage spécifiques si nécessaire :

```bash
# Configuration des ACLs personnalisées
ssh admin@firewall
sudo iptables -A FORWARD -s 10.10.X.X -d 192.168.Y.Y -p tcp --dport 3389 -j ACCEPT
sudo iptables-save > /etc/iptables/rules.v4
```

5. Mise à jour de la documentation et de l'inventaire des accès

**Communication à l'utilisateur :**

Une fois la configuration terminée, l'utilisateur reçoit :
- Son fichier de configuration OpenVPN pour chaque instance autorisée
- Ses identifiants pour l'authentification secondaire
- Un guide de première connexion
- Les coordonnées du support en cas de problème

## b. Révocation d'accès :

La révocation des accès VPN doit être effectuée rapidement lors du départ d'un collaborateur ou en cas de changement de fonction ne nécessitant plus certains accès.

**Procédure de révocation standard (départ planifié) :**

1. Réception de la notification de départ via le système RH

2. Planification de la révocation à la date de départ

3. Le jour du départ :
   - Révocation des certificats comme décrit précédemment
   - Désactivation du compte d'authentification :
   ```bash
   ssh admin@auth-server
   sudo passwd -l vpn-utilisateur
   ```
   - Suppression des règles de filtrage spécifiques le cas échéant

4. Mise à jour de la documentation et de l'inventaire des accès

**Procédure de révocation d'urgence :**

En cas de départ immédiat ou de suspicion de compromission, une procédure accélérée est mise en œuvre :

1. Révocation immédiate des certificats et déploiement de la CRL

2. Blocage au niveau du pare-feu des adresses IP attribuées à l'utilisateur :
```bash
ssh admin@firewall
sudo iptables -I FORWARD -s 10.10.X.X -j DROP
sudo iptables-save > /etc/iptables/rules.v4
```

3. Désactivation du compte d'authentification

4. Notification immédiate au responsable sécurité et à la DSI

## c. Modification des privilèges d'accès :

En cas de changement de fonction, les privilèges d'accès VPN peuvent nécessiter une modification.

**Procédure de modification d'accès :**

1. Réception de la demande validée par la hiérarchie

2. Analyse des nouveaux besoins d'accès

3. Modification des groupes d'appartenance :
```bash
# Ajout d'un accès développement
ssh admin@auth-server
sudo usermod -a -G vpn-dev vpn-utilisateur

# Suppression d'un accès production
sudo gpasswd -d vpn-utilisateur vpn-prod
```

4. Génération des nouveaux fichiers de configuration si nécessaire

5. Ajustement des règles de filtrage spécifiques

6. Mise à jour de la documentation et de l'inventaire des accès

7. Notification à l'utilisateur des changements effectués

# V. Surveillance et résolution d'incidents :
## a. Outils de monitoring :
