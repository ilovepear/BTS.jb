# Maintenance d'un VPN interne @ Oodrive

## Sommaire

### Nature de l'activité
- **Contexte**
- **Objectifs**

### Conditions de réalisations
- **Matériel**
- **Contraintes**
- **Logiciel**
- **Requis**
- **Difficultés rencontrées**
- **Durée de réalisation**

### Solutions envisageables

### Solution retenue
- **Conditions initiales**
- **Résultat final**
- **Outils utilisés**

### I. Introduction

### II. Présentation de l'infrastructure VPN
- **Architecture du réseau VPN**
- **Configuration des instances VPN**
- **Sécurisation des accès**
- **Certificats et authentification**
- **Journalisation et monitoring**

### III. Procédures de maintenance
- **Mise à jour du serveur VPN**
- **Gestion des certificats utilisateurs**

### IV. Gestion des accès utilisateurs
- **Création de compte utilisateur**
- **Révocation d'accès**
- **Modification des privilèges d'accès**

### V. Surveillance et résolution d'incidents
- **Outils de monitoring**
- **Procédures d'intervention**
- **Plan de reprise d'activité**

### VI. Documentation et conformité
- **Mise à jour de la documentation**
- **Audits de sécurité**

### VII. Conclusion

# Nature de l'activité :
## Contexte :
Dans le cadre de mon activité au sein d'Oodrive, entreprise certifiée SecNumCloud, je dois assurer la maintenance de l'infrastructure VPN interne permettant l'accès sécurisé aux ressources de l'entreprise pour les collaborateurs en télétravail et dans les différents bureaux. Cette infrastructure est critique car elle constitue le point d'entrée au réseau d'entreprise et doit répondre aux exigences strictes de la certification SecNumCloud.

## Objectifs :
Assurer le bon fonctionnement et la sécurité des trois instances VPN (Standard, développement et production), gérer les accès utilisateurs, effectuer les mises à jour de sécurité et maintenir la conformité avec les standards de sécurité requis par la certification SecNumCloud.

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
Infrastructure OpenVPN gérée en interne avec trois instances distinctes (Standard, développement, production) pour répondre aux besoins spécifiques d'accès aux ressources et garantir une séparation stricte des environnements.

### Conditions initiales :
Infrastructure VPN fonctionnelle mais nécessitant une maintenance régulière pour assurer sa conformité continue et son adaptation aux besoins évolutifs de l'entreprise.

### Résultat final :
Infrastructure VPN hautement disponible, sécurisée, conforme aux standards SecNumCloud et permettant un accès différencié aux ressources selon les profils utilisateurs.

### Outils utilisés :
OpenVPN, PuTTY, EasyRSA, OpenSSL, Zabbix, Elastic Stack, scripts bash personnalisés pour l'automatisation.

# I. Introduction :
Dans le cadre de mes responsabilités au sein de l'équipe IT d'Oodrive, j'assure la maintenance et la gestion de l'infrastructure VPN interne de l'entreprise. Cette infrastructure est particulièrement critique car Oodrive est certifiée SecNumCloud, certification française de haut niveau pour les fournisseurs de services cloud qui impose des exigences strictes en matière de sécurité et de disponibilité des systèmes.

Le VPN interne constitue la porte d'entrée sécurisée au réseau d'entreprise pour les collaborateurs en télétravail, dont le nombre a considérablement augmenté ces dernières années. Il permet également de sécuriser les communications entre les différents sites d'Oodrive et d'assurer un accès contrôlé aux ressources sensibles.

Pour répondre aux besoins spécifiques d'Oodrive en termes de séparation des environnements, l'infrastructure VPN est composée de trois instances distinctes :
- VPN Standard : pour l'accès aux ressources courantes de l'entreprise (bureautique, intranet, applications internes)
- VPN Développement : pour l'accès aux environnements de développement et de test
- VPN Production : pour l'accès aux environnements de production, hautement sécurisé et limité aux équipes opérationnelles

Cette organisation permet de respecter le principe de séparation des environnements imposé par la certification SecNumCloud tout en facilitant la gestion des accès selon les profils utilisateurs.

# II. Présentation de l'infrastructure VPN :
## a. Architecture du réseau VPN :

L'infrastructure VPN d'Oodrive repose sur une architecture distribuée avec trois serveurs dédiés, chacun hébergeant une instance spécifique du VPN. Ces serveurs sont situés dans le datacenter principal d'Oodrive et bénéficient des mesures de sécurité physique et logique associées.

Chaque instance VPN dispose de sa propre plage d'adresses IP privées pour éviter les conflits de routage :
- VPN Standard : 10.10.0.0/16
- VPN Développement : 10.20.0.0/16
- VPN Production : 10.30.0.0/16

Le schéma ci-dessous illustre l'architecture globale du réseau VPN :


Les connexions VPN utilisent le protocole OpenVPN en mode TLS avec une double authentification :
- Certificat client (authentification mutuelle TLS)
- Nom d'utilisateur et mot de passe (authentification secondaire)

Cette double authentification répond aux exigences de la certification SecNumCloud en matière d'authentification forte.

Les flux réseau sont contrôlés par des pare-feux dédiés qui filtrent les accès selon les règles définies pour chaque instance VPN :
- Le pare-feu d'entrée filtre les connexions entrantes vers les serveurs VPN
- Les pare-feux internes contrôlent les accès aux ressources de l'entreprise selon le profil de l'utilisateur et l'instance VPN utilisée

## b. Configuration des instances VPN :

Les trois instances VPN sont configurées de manière similaire mais avec des paramètres de sécurité et d'accès spécifiques à leur rôle.

**VPN Standard :**
- Port UDP 1194
- Chiffrement AES-256-GCM
- Compression LZ4
- MTU optimisé à 1400
- Authentification par certificat et credentials
- Accès aux ressources standard de l'entreprise
- Journalisation standard des connexions

Extrait de la configuration du serveur VPN Standard :
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
- Charge serveur aStandardment élevée
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
- Mises à jour de sécurité Standards : application hebdomadaire pendant une fenêtre de maintenance
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

Pour le VPN Standard :
```bash
./generate-config.sh utilisateur Standard > utilisateur-Standard.ovpn
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
scp crl.pem admin@vpn-Standard:/etc/openvpn/server/
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
   - Profil d'accès souhaité (Standard, développement, production)
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
# Pour accès VPN Standard
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

La surveillance continue de l'infrastructure VPN est essentielle pour garantir sa disponibilité et sa sécurité. Oodrive utilise une combinaison d'outils pour assurer un monitoring complet de l'environnement VPN.

**Zabbix :**

Zabbix est l'outil principal de monitoring utilisé pour la surveillance en temps réel des serveurs VPN. Il est configuré pour collecter et analyser divers indicateurs :

- Disponibilité des services OpenVPN (vérification toutes les 30 secondes)
- Métriques système : CPU, mémoire, espace disque, charge système
- Nombre de connexions VPN actives par instance
- Bande passante utilisée sur les interfaces réseau
- Temps de réponse des serveurs

Des seuils d'alerte sont configurés pour chaque métrique avec plusieurs niveaux de gravité :
- Information : notification simple sans action requise
- Avertissement : situation à surveiller
- Critique : intervention immédiate requise

Exemple de configuration d'un élément de monitoring dans Zabbix :
```
Name: OpenVPN Service Status
Key: service.info[openvpn@server]
Type: Zabbix agent
Update interval: 30s
```

**Elastic Stack :**

La suite Elastic Stack (Elasticsearch, Logstash, Kibana) est utilisée pour la collecte, l'indexation et l'analyse des logs générés par l'infrastructure VPN :

- Collecte des logs via Filebeat sur chaque serveur VPN
- Traitement et enrichissement via Logstash
- Indexation dans Elasticsearch
- Visualisation et analyse dans Kibana

Des tableaux de bord spécifiques sont configurés dans Kibana pour différents cas d'usage :
- Monitoring des connexions (succès/échecs)
- Analyse de sécurité (détection d'anomalies)
- Suivi de performance
- Audit de conformité

**Alertes et notifications :**

Un système d'alerte est configuré pour notifier l'équipe d'infrastructure en cas d'anomalie détectée :

- Alertes par email pour les problèmes non critiques
- Notifications SMS pour les incidents critiques
- Intégration avec le système de ticketing pour la création automatique d'incidents
- Escalade automatique vers l'astreinte en dehors des heures ouvrées

Exemple de règle d'alerte dans Zabbix :
```
Name: OpenVPN Service Down
Expression: {vpn-prod:service.info[openvpn@server].last()}=0
Severity: High
Actions: Send SMS to on-call team, Create incident ticket
```

## b. Procédures d'intervention :

Des procédures d'intervention standardisées sont définies pour faire face aux incidents les plus courants sur l'infrastructure VPN.

**Indisponibilité du service VPN :**

En cas de détection d'une indisponibilité du service VPN :

1. Vérification du statut du service sur le serveur concerné :
```bash
ssh admin@vpn-server
sudo systemctl status openvpn@server
```

2. Analyse des logs pour identifier la cause :
```bash
sudo tail -n 100 /var/log/openvpn/openvpn.log
sudo journalctl -u openvpn@server --since "10 minutes ago"
```

3. Redémarrage du service si nécessaire :
```bash
sudo systemctl restart openvpn@server
```

4. Vérification du bon fonctionnement après intervention :
```bash
sudo systemctl status openvpn@server
ping 10.10.0.1
```

5. Si le problème persiste, activation du serveur de secours et escalade vers le niveau 2

**Échecs d'authentification massifs :**

En cas de détection d'un nombre anormal d'échecs d'authentification :

1. Analyse des logs d'authentification pour identifier les sources :
```bash
grep "Auth failed" /var/log/openvpn/openvpn.log | tail -n 50
```

2. Vérification des tentatives par adresse IP :
```bash
grep "Auth failed" /var/log/openvpn/openvpn.log | awk '{print $9}' | sort | uniq -c | sort -nr
```

3. Si une attaque est suspectée, blocage temporaire des adresses IP concernées :
```bash
ssh admin@firewall
sudo iptables -I INPUT -s 203.0.113.x -p udp --dport 1194 -j DROP
sudo iptables-save > /etc/iptables/rules.v4
```

4. Notification au responsable sécurité pour analyse approfondie

**Problèmes de performance :**

En cas de ralentissements signalés par les utilisateurs :

1. Vérification de la charge système sur les serveurs VPN :
```bash
ssh admin@vpn-server
top -b -n 1
```

2. Analyse du nombre de connexions simultanées :
```bash
cat /var/log/openvpn/openvpn-status.log | grep "ROUTING TABLE" -A 500 | wc -l
```

3. Vérification de l'utilisation de la bande passante :
```bash
vnstat -h -i eth0
```

4. Si la charge est trop importante, mise en place de limitations de bande passante temporaires :
```bash
sudo tc qdisc add dev tun0 root tbf rate 5mbit burst 32kbit latency 50ms
```

5. Planification d'un renforcement des ressources si nécessaire

## c. Plan de reprise d'activité :

Un plan de reprise d'activité (PRA) est défini pour l'infrastructure VPN afin de garantir la continuité du service en cas d'incident majeur.

**Architecture de secours :**

Chaque instance VPN dispose d'un serveur de secours configuré en standby :
- Réplication régulière de la configuration
- Synchronisation des certificats et de la CRL
- Tests mensuels d'activation

La bascule vers les serveurs de secours peut être manuelle ou automatique selon la gravité de l'incident.

**Procédure de bascule :**

En cas d'incident nécessitant une bascule vers le serveur de secours :

1. Activation du serveur de secours :
```bash
ssh admin@vpn-backup
sudo cp /etc/openvpn/server/server.conf.standby /etc/openvpn/server/server.conf
sudo systemctl start openvpn@server
```

2. Modification des enregistrements DNS pour rediriger les connexions :
```bash
ssh admin@dns-server
sudo pdnsutil replace-rrset internal.oodrive.com vpn 300 A 192.168.1.x
```

3. Notification aux utilisateurs via le système d'alerte

4. Surveillance du bon fonctionnement du serveur de secours

**Procédure de restauration :**

Une fois l'incident résolu et le serveur principal réparé :

1. Synchronisation des données de session du serveur de secours vers le serveur principal
2. Validation du bon fonctionnement du serveur principal
3. Modification des enregistrements DNS pour rediriger les connexions vers le serveur principal
4. Notification aux utilisateurs via le système d'alerte
5. Surveillance du bon fonctionnement après bascule retour

**Tests de reprise :**

Des tests réguliers du PRA sont effectués pour garantir son efficacité :
- Tests mensuels de bascule manuelle planifiés pendant les fenêtres de maintenance
- Test annuel complet avec simulation d'incident majeur
- Analyse post-test pour identifier les axes d'amélioration

# VI. Documentation et conformité :
## a. Mise à jour de la documentation :

La documentation de l'infrastructure VPN est un élément essentiel pour sa maintenance et sa conformité avec les exigences SecNumCloud.

**Types de documentation maintenue :**

- Documentation technique :
  - Architecture détaillée de l'infrastructure VPN
  - Procédures d'installation et de configuration
  - Scripts et outils d'administration
  - Procédures de maintenance

- Documentation opérationnelle :
  - Procédures d'exploitation courante
  - Guides de résolution d'incidents
  - Calendrier des maintenances planifiées
  - Journal des interventions

- Documentation de sécurité :
  - Politique de sécurité VPN
  - Analyse de risques
  - Résultats des audits de sécurité
  - Procédures de gestion des incidents

- Documentation utilisateur :
  - Guides d'installation du client VPN
  - Procédures de connexion
  - FAQ et résolution des problèmes courants

**Processus de mise à jour :**

La documentation est maintenue à jour selon un processus formalisé :

1. Identification des modifications nécessaires lors de chaque intervention
2. Mise à jour de la documentation dans le système de gestion documentaire
3. Revue par un pair pour validation
4. Publication de la nouvelle version
5. Notification aux parties prenantes

Chaque document contient un historique des modifications avec :
- Date de modification
- Auteur
- Nature des changements
- Version du document

**Stockage et accès :**

La documentation est stockée dans une GED (Gestion Électronique des Documents) sécurisée avec :
- Contrôle des accès basé sur les rôles
- Versionnement des documents
- Historique des modifications
- Workflow d'approbation pour les documents critiques

## b. Audits de sécurité :

Des audits de sécurité réguliers sont réalisés pour vérifier la conformité de l'infrastructure VPN avec les exigences SecNumCloud et les bonnes pratiques de sécurité.

**Types d'audits réalisés :**

- Audits internes trimestriels par l'équipe sécurité d'Oodrive
- Audits externes annuels par un prestataire spécialisé
- Tests d'intrusion annuels ciblant spécifiquement l'infrastructure VPN
- Revue de configuration mensuelle

**Points de contrôle des audits :**

Les audits de sécurité couvrent les aspects suivants :
- Conformité de la configuration avec les recommandations ANSSI
- Vérification des niveaux de chiffrement
- Test des mécanismes d'authentification
- Analyse des journaux d'événements
- Vérification des procédures de gestion des certificats
- Test des mécanismes de filtrage réseau
- Vérification de l'application des correctifs de sécurité

**Gestion des non-conformités :**

En cas de détection d'une non-conformité lors d'un audit :

1. Évaluation de la criticité et de l'impact
2. Définition d'un plan de remédiation avec des échéances
3. Mise en œuvre des actions correctives
4. Vérification de l'efficacité des corrections
5. Mise à jour de la documentation

**Rapport de conformité SecNumCloud :**

Un rapport de conformité spécifique est maintenu pour les exigences SecNumCloud liées à l'infrastructure VPN :

| Exigence | Statut | Evidence | 
|----------|--------|----------|
| Chiffrement AES-256 | Conforme | Configuration OpenVPN |
| Double authentification | Conforme | Configuration auth |
| Journalisation complète | Conforme | Elastic Stack |
| Révocation certificats | Conforme | Procédure testée |
| Séparation des environnements | Conforme | Architecture réseau |

# VII. Conclusion :

La maintenance de l'infrastructure VPN d'Oodrive représente un enjeu crucial pour garantir l'accès sécurisé aux ressources de l'entreprise, particulièrement dans un contexte de télétravail accru et d'exigences strictes liées à la certification SecNumCloud.

Ce projet m'a permis de mettre en œuvre une approche structurée de la gestion d'une infrastructure critique, en abordant tous les aspects de son cycle de vie :
- Architecture sécurisée avec séparation des environnements
- Gestion rigoureuse des certificats et des accès
- Procédures de maintenance formalisées
- Monitoring et détection d'incidents
- Plan de reprise d'activité éprouvé
- Documentation complète et à jour
- Conformité continue avec les exigences de sécurité

L'infrastructure VPN mise en place répond pleinement aux exigences de la certification SecNumCloud tout en offrant une expérience utilisateur fluide pour les collaborateurs en télétravail. La séparation en trois instances distinctes (normale, développement, production) permet un contrôle fin des accès aux ressources selon les profils utilisateurs, renforçant ainsi la sécurité globale du système d'information d'Oodrive.

Les procédures de maintenance régulière et les mécanismes de monitoring mis en place permettent d'assurer une disponibilité optimale du service tout en garantissant sa sécurité. Le plan de reprise d'activité testé régulièrement assure quant à lui la continuité du service en cas d'incident majeur.

Cette expérience a renforcé mes compétences en gestion d'infrastructures critiques et en sécurisation des accès distants, des domaines particulièrement stratégiques dans le contexte actuel où le télétravail est devenu une norme pour de nombreuses entreprises.
