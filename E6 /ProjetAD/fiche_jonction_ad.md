# Fiche Utilisateur - Jonction Ubuntu Desktop au domaine Active Directory

## 📋 Informations générales

**Objectif** : Intégrer une machine Ubuntu Desktop dans le domaine Active Directory `hn.gua.local`

**Configuration cible** :
- Machine Ubuntu Desktop : `192.168.10.20`
- Domaine AD : `hn.gua.local`
- Contrôleur de domaine : `192.168.10.1`
- Utilisateur local : `vboxuser`

---

## 🔧 Prérequis et vérifications

### 1. Vérification de la connectivité réseau

Avant de commencer, vérifiez que votre machine peut communiquer avec le contrôleur de domaine :

```bash
# Test de connectivité
ping 192.168.10.1

# Vérification de la résolution DNS
nslookup hn.gua.local 192.168.10.1
```

### 2. Configuration VirtualBox requise

Votre machine virtuelle doit avoir :
- **Carte réseau 1** : Réseau interne (intnet) - pour la communication AD
- **Carte réseau 2** : NAT - pour l'accès Internet temporaire

---

## 🛠️ Étape 1 - Configuration réseau Ubuntu

### Configuration des interfaces réseau

**Important** : Cette configuration doit être appliquée à chaque démarrage de la machine.

```bash
# Configuration de l'interface réseau interne (enp0s3)
sudo ip addr flush dev enp0s3
sudo ip addr add 192.168.10.20/24 dev enp0s3
sudo ip link set enp0s3 up

# Configuration DNS pour le domaine AD
sudo resolvectl dns enp0s3 192.168.10.1
sudo resolvectl domain enp0s3 hn.gua.local
```

### Configuration du fichier resolv.conf

```bash
# Sauvegarder le fichier resolv.conf original
sudo cp /etc/resolv.conf /etc/resolv.conf.backup

# Configurer resolv.conf pour le domaine AD
sudo tee /etc/resolv.conf > /dev/null << EOF
# Configuration DNS pour le domaine Active Directory
nameserver 192.168.10.1
search hn.gua.local
domain hn.gua.local
EOF

# Protéger le fichier contre les modifications automatiques
sudo chattr +i /etc/resolv.conf
```

### Activation temporaire d'Internet (si nécessaire)

```bash
# Pour les mises à jour de paquets - temporairement déprotéger resolv.conf
sudo chattr -i /etc/resolv.conf

# Configurer temporairement pour Internet
sudo dhclient enp0s8
sudo resolvectl dns enp0s8 8.8.8.8

# Après installation des paquets, remettre la config AD
sudo tee /etc/resolv.conf > /dev/null << EOF
nameserver 192.168.10.1
search hn.gua.local
domain hn.gua.local
EOF
sudo chattr +i /etc/resolv.conf
```

### Vérification de la configuration

```bash
# Vérifier les adresses IP
ip addr show

# Vérifier la configuration DNS
resolvectl status

# Vérifier le contenu de resolv.conf
cat /etc/resolv.conf

# Tester la résolution du domaine
nslookup hn.gua.local
nslookup _ldap._tcp.hn.gua.local

# Vérifier la résolution inverse
nslookup 192.168.10.1
```

---

## 📦 Étape 2 - Installation des paquets requis

### Installation des outils de jonction AD

```bash
sudo apt update && sudo apt install -y \
    realmd \
    sssd \
    sssd-tools \
    libnss-sss \
    libpam-sss \
    adcli \
    samba-common-bin \
    oddjob \
    oddjob-mkhomedir \
    packagekit \
    krb5-user \
    openssh-server \
    dnsutils \
    net-tools
```

**Note** : L'installation peut prendre plusieurs minutes. Assurez-vous d'avoir une connexion Internet active.

---

## ⏰ Étape 3 - Synchronisation temporelle

La synchronisation avec le contrôleur de domaine est cruciale :

```bash
# Activer la synchronisation NTP
sudo timedatectl set-ntp true

# Vérifier l'heure système
timedatectl status
```

---

## 🔍 Étape 4 - Découverte du domaine

### Vérification de la découverte du domaine

```bash
# Découvrir le domaine AD
sudo realm discover hn.gua.local
```

**Résultat attendu** :
```
hn.gua.local
  type: kerberos
  realm-name: HN.GUA.LOCAL
  domain-name: hn.gua.local
  configured: no
  server-software: active-directory
  client-software: sssd
  required-package: sssd-tools
  required-package: sssd
  required-package: libnss-sss
  required-package: libpam-sss
  required-package: adcli
  required-package: samba-common-bin
```

---

## 🏗️ Étape 5 - Configuration SSSD

### Création du fichier de configuration SSSD

```bash
# Créer le fichier de configuration
sudo tee /etc/sssd/sssd.conf > /dev/null << EOF
[sssd]
domains = hn.gua.local
config_file_version = 2
services = nss, pam

[domain/hn.gua.local]
default_shell = /bin/bash
krb5_store_password_if_offline = True
cache_credentials = True
krb5_realm = HN.GUA.LOCAL
realmd_tags = manages-system joined-with-adcli
id_provider = ad
fallback_homedir = /home/%u
ad_domain = hn.gua.local
use_fully_qualified_names = False
ldap_id_mapping = True
access_provider = ad
ad_gpo_access_control = permissive
EOF

# Sécuriser le fichier de configuration
sudo chmod 600 /etc/sssd/sssd.conf
sudo chown root:root /etc/sssd/sssd.conf
```

---

## 🔗 Étape 6 - Jonction au domaine

### Rejoindre le domaine Active Directory

```bash
# Joindre le domaine avec le compte Administrateur
sudo realm join --user=Administrateur hn.gua.local
```

**Attention** : Vous serez invité à saisir le mot de passe du compte `Administrateur` du domaine.

### Vérification de la jonction

```bash
# Vérifier que la machine a rejoint le domaine
sudo realm list

# Vérifier l'état SSSD
sudo systemctl status sssd
```

---

## ⚙️ Étape 7 - Configuration post-jonction et gestion des sessions

### Autoriser tous les utilisateurs du domaine

```bash
# Permettre à tous les utilisateurs du domaine de se connecter
sudo realm permit --all
```

### Configuration PAM pour la création automatique des répertoires home

```bash
# Activer la création automatique des dossiers utilisateurs
sudo pam-auth-update --enable mkhomedir

# Vérifier que pam_mkhomedir est bien configuré
grep mkhomedir /etc/pam.d/common-session

# Éditer manuellement le fichier common-session si nécessaire
sudo nano /etc/pam.d/common-session

# Ajouter cette ligne si elle n'existe pas (à la fin du fichier) :
# session optional pam_mkhomedir.so skel=/etc/skel umask=022

# Vérifier également le fichier common-account
sudo nano /etc/pam.d/common-account

# S'assurer que cette ligne est présente :
# account [success=1 new_authtok_reqd=done default=ignore] pam_unix.so
```

**Contenu attendu dans `/etc/pam.d/common-session` :**
```
# Ligne à ajouter pour la création automatique des répertoires home
session optional pam_mkhomedir.so skel=/etc/skel umask=022
```

### Configuration des sessions utilisateur AD

```bash
# S'assurer que les utilisateurs AD ont un shell par défaut
sudo tee -a /etc/sssd/sssd.conf > /dev/null << 'EOF'

# Configuration shell par défaut pour les utilisateurs AD
override_shell = /bin/bash
EOF

# Configurer la création des répertoires home avec les bonnes permissions
sudo mkdir -p /etc/skel
sudo chmod 755 /etc/skel

# Configurer les permissions par défaut pour les nouveaux utilisateurs
echo "umask 022" | sudo tee -a /etc/skel/.bashrc
```

### Redémarrage des services

```bash
# Redémarrer les services nécessaires
sudo systemctl restart sssd
sudo systemctl restart oddjobd
sudo systemctl enable sssd
sudo systemctl enable oddjobd
```

---

## ✅ Étape 8 - Tests et vérifications des sessions utilisateur

### Test de résolution des utilisateurs AD

```bash
# Tester la résolution d'un utilisateur du domaine
getent passwd j.boungo

# Lister les utilisateurs du domaine
getent passwd | grep -v "^#"
```

### Test d'authentification et création de session propre

```bash
# Méthode 1 : Création d'une session interactive complète
sudo -u j.boungo -i

# Méthode 2 : Session SSH locale (recommandée pour les tests)
ssh j.boungo@localhost

# Méthode 3 : Switch user avec environnement complet
su - j.boungo
```

### Vérification de l'environnement utilisateur

Une fois connecté en tant que `j.boungo`, vérifiez :

```bash
# Vérifier l'identité de l'utilisateur
whoami
id

# Vérifier le répertoire home
pwd
ls -la

# Vérifier les variables d'environnement
echo $HOME
echo $USER
echo $SHELL

# Tester l'accès aux ressources du domaine
klist  # Vérifier les tickets Kerberos
```

### Vérification des services

```bash
# Vérifier l'état des services critiques
sudo systemctl status sssd
sudo systemctl status oddjobd
sudo systemctl status ssh

# Vérifier les logs de connexion
sudo journalctl -u sssd -n 20
```

### Test de reconnexion automatique

```bash
# Sortir de la session utilisateur
exit

# Se reconnecter pour vérifier la persistence
ssh j.boungo@localhost
# ou
sudo -u j.boungo -i
```

---

## 🔧 Dépannage courant

### Problème : Résolution DNS ne fonctionne pas

**Solution** :
```bash
# Vérifier le fichier resolv.conf
cat /etc/resolv.conf

# Reconfigurer resolv.conf si nécessaire
sudo chattr -i /etc/resolv.conf
sudo tee /etc/resolv.conf > /dev/null << EOF
nameserver 192.168.10.1
search hn.gua.local
domain hn.gua.local
EOF
sudo chattr +i /etc/resolv.conf

# Redémarrer le service réseau
sudo systemctl restart systemd-resolved

# Tester la résolution
nslookup hn.gua.local
```

### Problème : Échec de la jonction au domaine

**Solutions possibles** :
1. Vérifier l'heure système (doit être synchronisée)
2. Vérifier les credentials de l'administrateur
3. Contrôler la connectivité réseau

```bash
# Test de connectivité Kerberos
kinit Administrateur@HN.GUA.LOCAL
klist
```

### Problème : Utilisateur AD non trouvé après jonction

**Solution** :
```bash
# Vider le cache SSSD et redémarrer
sudo sss_cache -E
sudo systemctl restart sssd
```

### Problème : PAM ne crée pas les répertoires home automatiquement

**Solution** :
```bash
# Vérifier la configuration PAM
grep mkhomedir /etc/pam.d/common-session

# Éditer manuellement si nécessaire
sudo nano /etc/pam.d/common-session

# Ajouter cette ligne à la fin du fichier :
# session optional pam_mkhomedir.so skel=/etc/skel umask=022

# Vérifier le fichier common-account
sudo nano /etc/pam.d/common-account

# Redémarrer oddjobd après modification
sudo systemctl restart oddjobd
```

---

## 📝 Configuration persistante du réseau et DNS

Pour éviter de reconfigurer le réseau à chaque démarrage, créez un script de démarrage :

```bash
# Créer un script de configuration réseau et DNS
sudo tee /etc/systemd/system/configure-network.service > /dev/null << EOF
[Unit]
Description=Configure Network for AD
After=network.target

[Service]
Type=oneshot
ExecStart=/bin/bash -c 'ip addr flush dev enp0s3; ip addr add 192.168.10.20/24 dev enp0s3; ip link set enp0s3 up; resolvectl dns enp0s3 192.168.10.1; resolvectl domain enp0s3 hn.gua.local'
ExecStartPost=/bin/bash -c 'chattr -i /etc/resolv.conf; echo "nameserver 192.168.10.1" > /etc/resolv.conf; echo "search hn.gua.local" >> /etc/resolv.conf; echo "domain hn.gua.local" >> /etc/resolv.conf; chattr +i /etc/resolv.conf'
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
EOF

# Activer le service
sudo systemctl enable configure-network.service

# Créer également un script manuel pour la configuration
sudo tee /usr/local/bin/configure-ad-network.sh > /dev/null << 'EOF'
#!/bin/bash
# Script de configuration réseau pour AD

echo "Configuration du réseau pour Active Directory..."

# Configuration interface
ip addr flush dev enp0s3
ip addr add 192.168.10.20/24 dev enp0s3
ip link set enp0s3 up

# Configuration DNS
resolvectl dns enp0s3 192.168.10.1
resolvectl domain enp0s3 hn.gua.local

# Configuration resolv.conf
chattr -i /etc/resolv.conf 2>/dev/null || true
cat > /etc/resolv.conf << EOL
nameserver 192.168.10.1
search hn.gua.local
domain hn.gua.local
EOL
chattr +i /etc/resolv.conf

echo "Configuration réseau AD terminée."
EOF

# Rendre le script exécutable
sudo chmod +x /usr/local/bin/configure-ad-network.sh

# Pour exécuter manuellement la configuration :
# sudo /usr/local/bin/configure-ad-network.sh
```

---

## 🎯 Validation finale - Sessions utilisateur

Une fois toutes les étapes terminées, vous devriez pouvoir :

1. **Créer une session complète avec un utilisateur du domaine** :
   ```bash
   sudo -u j.boungo -i
   # ou
   ssh j.boungo@localhost
   ```

2. **Vérifier l'environnement utilisateur** :
   ```bash
   # Dans la session j.boungo
   whoami  # Doit retourner : j.boungo
   pwd     # Doit retourner : /home/j.boungo
   id      # Doit montrer les groupes AD
   ```

3. **Accéder aux ressources réseau** du domaine avec authentification automatique

4. **Bénéficier des GPO** appliquées par le contrôleur de domaine

### Commandes de vérification finale des sessions

```bash
# Vérifier la jonction au domaine
sudo realm list

# Tester la création d'une session utilisateur complète
sudo -u j.boungo -i -c "whoami && pwd && id"

# Vérifier les services
sudo systemctl is-active sssd oddjobd

# Test de connectivité AD avec l'utilisateur
sudo -u j.boungo kinit
sudo -u j.boungo klist

# Vérifier les permissions sur le répertoire home
ls -la /home/ | grep j.boungo
```

---

## 📞 Support et ressources

En cas de problème persistant, consultez :
- Les logs SSSD : `/var/log/sssd/`
- Les logs système : `journalctl -u sssd`
- La documentation officielle Ubuntu pour l'intégration AD

**Contact support** : [Indiquer ici les informations de contact du support technique]