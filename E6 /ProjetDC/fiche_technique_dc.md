# 1. INFORMATIONS GÉNÉRALES

## Spécifications du serveur
- **Adresse IP** : 192.168.10.1
- **Nom FQDN** : dc1.hn.gua.local
- **Domaine AD** : hn.gua.local
- **Système d'exploitation** : Windows Server 2019/2022
- **Rôles installés** : 
  - Active Directory Domain Services (AD DS)
  - DNS Server
  - Group Policy Management

![Capture d'écran 2025-04-11 155147](https://github.com/user-attachments/assets/38444425-b479-4c6a-8544-57323c16116e)

- **Sécurité réseau** : UFW (Uncomplicated Firewall)

## Architecture réseau
- **Réseau interne** : 192.168.10.0/24
- **Passerelle** : 192.168.10.1 (le serveur lui-même)
- **Serveur DNS primaire** : 192.168.10.1
- **Serveur DNS secondaire** : 8.8.8.8 (fallback)

# 2. CONFIGURATION ACTIVE DIRECTORY DOMAIN SERVICES

## Installation et promotion
```powershell
# Installation du rôle AD DS
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools

# Promotion en contrôleur de domaine
Install-ADDSForest `
  -DomainName "hn.gua.local" `
  -DomainNetbiosName "HN" `
  -InstallDns:$true `
  -LogPath "C:\Windows\NTDS" `
  -NoRebootOnCompletion:$false `
  -SysvolPath "C:\Windows\SYSVOL" `
  -Force:$true
```

Equivalent graphique :

![Capture d'écran 2025-04-11 154555](https://github.com/user-attachments/assets/576247b5-fc94-4ca1-9c4b-f1f2b260a129)
![Capture d'écran 2025-04-11 145836](https://github.com/user-attachments/assets/cffac97c-586e-47eb-9226-3982caa7ae7d)
![Capture d'écran 2025-04-11 145812](https://github.com/user-attachments/assets/871ecbd4-b1f7-4c88-98a7-1e6802629a21)
![Capture d'écran 2025-04-11 145755](https://github.com/user-attachments/assets/251e8bfb-e991-4662-ad2f-7f1ac0aa90f6)
![Capture d'écran 2025-04-11 145723](https://github.com/user-attachments/assets/4cf5e26c-acf0-46f1-96a2-228b7674cb5f)
![Capture d'écran 2025-04-11 142610](https://github.com/user-attachments/assets/1c0a9ca7-9cd1-4741-ad0e-f2bfe80f434a)



## Structure organisationnelle
```
hn.gua.local
├── Builtin
├── Computers
├── Domain Controllers
├── ForeignSecurityPrincipals
├── Managed Service Accounts
├── Users
└── Custom OUs
    ├── Workstations
    │   ├── Ubuntu_Machines
    │   └── Windows_Machines
    ├── Users_Accounts
    │   ├── Admins
    │   ├── Standard_Users
    │   └── Service_Accounts
    └── Groups
        ├── Security_Groups
        └── Distribution_Groups
```

## Paramètres du domaine
- **Niveau fonctionnel du domaine** : Windows Server 2016 minimum
- **Niveau fonctionnel de la forêt** : Windows Server 2016 minimum
- **Mode de réplication** : FRS ou DFSR (recommandé)
- **Catalogue global** : Activé sur le contrôleur principal

## Services critiques AD
```powershell
# Vérification des services AD essentiels
Get-Service | Where-Object {$_.Name -in @('ADWS','DNS','KDC','Netlogon','NTDS','W32Time')} | Format-Table Name,Status,StartType
```

Services requis :
- **ADWS** (Active Directory Web Services)
- **DNS** (Serveur DNS)
- **KDC** (Kerberos Key Distribution Center)
- **Netlogon** (Service d'ouverture de session réseau)
- **NTDS** (Active Directory Domain Services)
- **W32Time** (Service de temps Windows)

# 3. CONFIGURATION DNS

## Zones DNS principales
```powershell
# Zone de recherche directe
Add-DnsServerPrimaryZone -Name "hn.gua.local" -ZoneFile "hn.gua.local.dns"

# Zone de recherche inversée
Add-DnsServerPrimaryZone -NetworkID "192.168.10.0/24" -ZoneFile "10.168.192.in-addr.arpa.dns"
```

## Enregistrements DNS critiques
```
# Enregistrements SRV requis pour AD
_ldap._tcp.hn.gua.local.        SRV 0 100 389 dc1.hn.gua.local.
_kerberos._tcp.hn.gua.local.    SRV 0 100 88  dc1.hn.gua.local.
_gc._tcp.hn.gua.local.          SRV 0 100 3268 dc1.hn.gua.local.
_kpasswd._tcp.hn.gua.local.     SRV 0 100 464 dc1.hn.gua.local.

# Enregistrements A
dc1.hn.gua.local.               A   192.168.10.1
hn.gua.local.                   A   192.168.10.1

# Enregistrement PTR
1.10.168.192.in-addr.arpa.      PTR dc1.hn.gua.local.
```

## Configuration DNS avancée
```powershell
# Activation de la mise à jour dynamique sécurisée
Set-DnsServerPrimaryZone -Name "hn.gua.local" -DynamicUpdate "Secure"

# Configuration des redirecteurs DNS
Add-DnsServerForwarder -IPAddress "8.8.8.8","8.8.4.4"

# Activation du nettoyage automatique des enregistrements obsolètes
Set-DnsServerScavenging -ScavengingState $true -ScavengingInterval 7.00:00:00
Set-DnsServerZoneAging -Name "hn.gua.local" -Aging $true -RefreshInterval 7.00:00:00 -NoRefreshInterval 7.00:00:00
```

## Vérification DNS
```powershell
# Test de résolution DNS
nslookup hn.gua.local 192.168.10.1
nslookup dc1.hn.gua.local 192.168.10.1

# Vérification des enregistrements SRV
nslookup -type=SRV _ldap._tcp.hn.gua.local 192.168.10.1
```

# 4. CONFIGURATION GROUP POLICY OBJECTS (GPO)
## Objectif :
Mettre en place les politiques de groupe nécessaires à la gestion des utilisateurs dans le domaine `hn.gua.local`, suite à la restauration du contrôleur de domaine `SRV-DC-GUA`.

---

## GPO 1 – Politique ComplexPassword

**Lieu** : Configuration par défaut du domaine  

**Chemin** :  
```
Configuration ordinateur > Paramètres Windows > Paramètres de sécurité > Stratégies de compte > Stratégie de mot de passe
```

- Mot de passe complexe : Activé  
- Longueur minimale : 8 caractères  
- Durée de vie maximale : 90 jours (modifiable)

---

## GPO 2 – Wallpaper

**Nom GPO** : `GPO_Wallpaper_Personnel`  
**Lieu** : Configuration par défaut du domaine 

**Chemin** :  
```
Configuration utilisateur > Stratégies > Modèles d'administration > Bureau > Bureau > Papier peint du bureau
```

- Activer l’option  
- Chemin : `\\SRV-DC-GUA\Downloads\wallpaper.jpg`  
- Style : Ajuster

---

## GPO 3 – AutoLockNoGDT

**Nom GPO** : `GPO_AutoLock`  
**Lieu** : OU globale, Groupe IT exclu

**Chemin** :  
```
Configuration utilisateur > Modèles d’administration > Panneau de configuration > Personnalisation
```

- Activer l’écran de verrouillage après inactivité  
- Délai : 600 secondes

---

## Application et filtrage

- Chaque GPO est liée au domaine en général
- Le filtrage par groupe de sécurité est utilisé pour appliquer certaines GPO uniquement à `Médecins`, `Infirmiers`, ou `IT`

- Vérification avec :  
```cmd
gpresult /r /scope:user
```

## Vérification et maintenance des GPO
```powershell
# Génération du rapport GPO
Get-GPOReport -All -ReportType HTML -Path "C:\GPO_Reports\All_GPOs_Report.html"

# Vérification des liens GPO
Get-GPInheritance -Target "OU=Workstations,DC=hn,DC=gua,DC=local"

# Forcer la mise à jour des GPO sur le serveur
Invoke-GPUpdate -Computer "dc1.hn.gua.local" -Force
```

# 5. CONFIGURATION FIREWALL (NetFIREWALL)
## ÉTAPE 1 : Configuration de Base Sécurisée
```powershell
# Principe du moindre privilège - Tout est bloqué par défaut
Set-NetFirewallProfile -Profile Domain,Public,Private -DefaultInboundAction Block -DefaultOutboundAction Block

Write-Host "✅ Configuration restrictive appliquée - Conformité HIPAA"
```
**Pourquoi cette étape ?**
En environnement médical, la protection des données patients (ePHI) est cruciale. Le principe du "moindre privilège" impose de bloquer tout trafic non nécessaire.

## ÉTAPE 2 : Services d'Authentification Active Directory
```powershell
# DNS - Résolution de noms (Port 53)
New-NetFirewallRule -DisplayName "DNS TCP" -Direction Inbound -Protocol TCP -LocalPort 53 -Action Allow -Profile Domain
New-NetFirewallRule -DisplayName "DNS UDP" -Direction Inbound -Protocol UDP -LocalPort 53 -Action Allow -Profile Domain

# Kerberos - Authentification sécurisée (Port 88)
New-NetFirewallRule -DisplayName "Kerberos TCP" -Direction Inbound -Protocol TCP -LocalPort 88 -Action Allow -Profile Domain
New-NetFirewallRule -DisplayName "Kerberos UDP" -Direction Inbound -Protocol UDP -LocalPort 88 -Action Allow -Profile Domain

# LDAP - Annuaire utilisateurs (Port 389/636)
New-NetFirewallRule -DisplayName "LDAP TCP" -Direction Inbound -Protocol TCP -LocalPort 389 -Action Allow -Profile Domain
New-NetFirewallRule -DisplayName "LDAPS" -Direction Inbound -Protocol TCP -LocalPort 636 -Action Allow -Profile Domain
```
**Explication technique :**
- **DNS** : Permet la résolution des noms d'ordinateurs en adresses IP
- **Kerberos** : Protocole d'authentification sécurisé d'Active Directory
- **LDAP/LDAPS** : Accès à l'annuaire AD (LDAPS = version chiffrée)

## ÉTAPE 3 : Sécurisation Réseau (Restriction Géographique)
```powershell
# Restriction LDAP au réseau interne uniquement
New-NetFirewallRule -DisplayName "LDAP Réseau Interne" -Direction Inbound -Protocol TCP -LocalPort 389 -RemoteAddress 192.168.10.0/24 -Action Allow -Profile Domain

# Blocage réseaux externes
New-NetFirewallRule -DisplayName "Blocage 10.x" -Direction Inbound -RemoteAddress 10.0.0.0/8 -Action Block
New-NetFirewallRule -DisplayName "Blocage 172.16.x" -Direction Inbound -RemoteAddress 172.16.0.0/12 -Action Block
```
**Principe de défense en profondeur :**
On limite l'accès aux services critiques au réseau interne uniquement. Cela protège contre les intrusions externes.

## ÉTAPE 4 : Services Windows pour Active Directory
```powershell
# RPC Endpoint Mapper (Port 135) - Communication AD
New-NetFirewallRule -DisplayName "RPC Endpoint" -Direction Inbound -Protocol TCP -LocalPort 135 -Action Allow -Profile Domain

# SMB (Port 445) - Partage SYSVOL/NETLOGON
New-NetFirewallRule -DisplayName "SMB SYSVOL" -Direction Inbound -Protocol TCP -LocalPort 445 -RemoteAddress 192.168.10.0/24 -Action Allow -Profile Domain

# Changement mot de passe Kerberos (Port 464)
New-NetFirewallRule -DisplayName "Kerberos Password TCP" -Direction Inbound -Protocol TCP -LocalPort 464 -Action Allow -Profile Domain
New-NetFirewallRule -DisplayName "Kerberos Password UDP" -Direction Inbound -Protocol UDP -LocalPort 464 -Action Allow -Profile Domain

# Global Catalog (Ports 3268/3269) - Recherche forêt AD
New-NetFirewallRule -DisplayName "Global Catalog" -Direction Inbound -Protocol TCP -LocalPort 3268 -Action Allow -Profile Domain
New-NetFirewallRule -DisplayName "Global Catalog SSL" -Direction Inbound -Protocol TCP -LocalPort 3269 -Action Allow -Profile Domain
```
**Rôle de chaque service :**
- **RPC** : Communication entre services Windows
- **SMB** : Partage des politiques de groupe (GPO)
- **Global Catalog** : Recherche dans toute la forêt AD (environnements multi-domaines)

## ÉTAPE 5 : Administration Sécurisée
```powershell
# RDP limité à l'administrateur IT
New-NetFirewallRule -DisplayName "RDP Admin" -Direction Inbound -Protocol TCP -LocalPort 3389 -RemoteAddress 192.168.10.100 -Action Allow -Profile Domain
```
**Bonnes pratiques :** L'accès RDP est restreint à une seule machine d'administration pour limiter les risques.

## ÉTAPE 6 : Trafic Sortant Autorisé
```powershell
# DNS sortant pour résolution externe
New-NetFirewallRule -DisplayName "DNS Sortant" -Direction Outbound -Protocol UDP -RemotePort 53 -Action Allow

# HTTPS pour mises à jour Windows
New-NetFirewallRule -DisplayName "HTTPS Updates" -Direction Outbound -Protocol TCP -RemotePort 443 -Action Allow

# NTP pour synchronisation temps (crucial pour logs d'audit)
New-NetFirewallRule -DisplayName "NTP Sync" -Direction Outbound -Protocol UDP -RemotePort 123 -Action Allow
```
**Pourquoi autoriser le trafic sortant ?**
- Mises à jour de sécurité Windows essentielles
- Synchronisation temporelle pour la traçabilité HIPAA

## ÉTAPE 7 : Journalisation et Audit (Exigence HIPAA)
```powershell
# Journalisation complète pour audit de conformité
Set-NetFirewallProfile -Profile Domain,Public,Private -LogAllowed True -LogBlocked True -LogMaxSizeKilobytes 8192

# Audit des connexions utilisateurs
auditpol /set /subcategory:"Logon" /success:enable /failure:enable
auditpol /set /subcategory:"Account Logon" /success:enable /failure:enable

Write-Host "✅ Audit configuré - Conformité HIPAA 164.312(b)"
```
**Importance de l'audit :** HIPAA impose la traçabilité de tous les accès aux données patients.

## ÉTAPE 8 : Sécurisation Préventive
```powershell
# Blocage des protocoles non sécurisés
New-NetFirewallRule -DisplayName "Blocage Telnet" -Direction Inbound -Protocol TCP -LocalPort 23 -Action Block
New-NetFirewallRule -DisplayName "Blocage FTP" -Direction Inbound -Protocol TCP -LocalPort 21 -Action Block
New-NetFirewallRule -DisplayName "Blocage SNMP" -Direction Inbound -Protocol UDP -LocalPort 161 -Action Block
```
**Principe de sécurité :** Bloquer explicitement les protocoles dangereux (non chiffrés).


# 6. MONITORING ET MAINTENANCE (PRODUCTION)

## Surveillance en temps réel
```powershell
# Installation de System Center Operations Manager (SCOM) ou équivalent
# Configuration des alertes critiques

# Métriques à surveiller en continu :
# - Santé des contrôleurs de domaine
# - Réplication AD (latence < 15 minutes)
# - Utilisation CPU (< 70% en moyenne)
# - Utilisation RAM (< 80%)
# - Espace disque libre (> 20%)
# - Temps de réponse LDAP (< 50ms)
# - Événements de sécurité critiques

# Script de surveillance automatisée
$HealthCheck = @"
# Vérification complète de la santé AD
dcdiag /v /c /d /e /s:dc1.hn.gua.local /f:C:\Logs\DCDiag_DC1.log
dcdiag /v /c /d /e /s:dc2.hn.gua.local /f:C:\Logs\DCDiag_# Fiche Technique - Serveur Active Directory avec DNS, GPO et UFW

# 1. INFORMATIONS GÉNÉRALES

## Spécifications du serveur (Production)
- **Serveur Primaire** : dc1.hn.gua.local (192.168.10.1)
- **Serveur Secondaire** : dc2.hn.gua.local (192.168.10.2) - **OBLIGATOIRE pour la production**
- **Domaine AD** : hn.gua.local
- **Système d'exploitation** : Windows Server 2022 (dernière version recommandée)
- **Configuration matérielle minimale** :
  - CPU : 4 cœurs minimum (8 recommandés)
  - RAM : 8 Go minimum (16-32 Go recommandés)
  - Stockage : SSD 500 Go minimum (RAID 1 obligatoire)
  - Réseau : 2 cartes réseau (redondance)
- **Rôles installés** : 
  - Active Directory Domain Services (AD DS)
  - DNS Server
  - Group Policy Management
  - Windows Server Backup
  - Active Directory Certificate Services (ADCS) - **Recommandé pour la production**
- **Sécurité réseau** : Windows Defender Firewall + Pare-feu périmétrique

## Architecture réseau
- **Réseau interne** : 192.168.10.0/24
- **Passerelle** : 192.168.10.1 (le serveur lui-même)
- **Serveur DNS primaire** : 192.168.10.1
- **Serveur DNS secondaire** : 8.8.8.8 (fallback)

# 2. CONFIGURATION ACTIVE DIRECTORY DOMAIN SERVICES

## Installation et promotion (Production)
```powershell
# Installation du rôle AD DS sur DC1 (Contrôleur principal)
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools

# Promotion en contrôleur de domaine principal
Install-ADDSForest `
  -DomainName "hn.gua.local" `
  -DomainNetbiosName "HN" `
  -InstallDns:$true `
  -DatabasePath "D:\Windows\NTDS" `
  -LogPath "E:\Windows\NTDS" `
  -SysvolPath "D:\Windows\SYSVOL" `
  -DomainMode "WinThreshold" `
  -ForestMode "WinThreshold" `
  -NoRebootOnCompletion:$false `
  -Force:$true

# Installation du second contrôleur de domaine (DC2) - OBLIGATOIRE
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools

# Promotion du second contrôleur
Install-ADDSDomainController `
  -DomainName "hn.gua.local" `
  -InstallDns:$true `
  -DatabasePath "D:\Windows\NTDS" `
  -LogPath "E:\Windows\NTDS" `
  -SysvolPath "D:\Windows\SYSVOL" `
  -NoRebootOnCompletion:$false `
  -Force:$true
```

## Haute disponibilité et redondance
```powershell
# Configuration du second DC comme serveur de catalogue global
Set-ADDomainController -Identity "dc2.hn.gua.local" -GlobalCatalog $true

# Configuration des maîtres d'opérations (FSMO) - Répartition recommandée
# DC1 : Schema Master, Domain Naming Master, RID Master
# DC2 : PDC Emulator, Infrastructure Master

Move-ADDirectoryServerOperationMasterRole -Identity "dc2.hn.gua.local" -OperationMasterRole PDCEmulator,InfrastructureMaster

# Vérification de la réplication
Get-ADReplicationConnection -Filter *
repadmin /replsummary
```

## Structure organisationnelle (Production)
```
hn.gua.local
├── Builtin
├── Computers
├── Domain Controllers
├── ForeignSecurityPrincipals
├── Managed Service Accounts
├── Users
└── Production OUs
    ├── Servers
    │   ├── Critical_Servers
    │   ├── Application_Servers
    │   └── File_Servers
    ├── Workstations
    │   ├── Executive_Workstations
    │   ├── Standard_Workstations
    │   ├── Ubuntu_Machines
    │   └── Quarantine
    ├── Users_Accounts
    │   ├── Domain_Admins
    │   ├── Server_Admins
    │   ├── Helpdesk
    │   ├── Standard_Users
    │   ├── Service_Accounts
    │   ├── Contractors
    │   └── Disabled_Accounts
    ├── Groups
    │   ├── Security_Groups
    │   │   ├── Admin_Groups
    │   │   ├── Resource_Access_Groups
    │   │   └── Application_Groups
    │   └── Distribution_Groups
    └── Resources
        ├── Shared_Folders
        ├── Printers
        └── Applications
```

## Paramètres du domaine (Production)
- **Niveau fonctionnel du domaine** : Windows Server 2022 (dernier niveau)
- **Niveau fonctionnel de la forêt** : Windows Server 2022 (dernier niveau)
- **Mode de réplication** : DFSR (Distribution File System Replication) - **Obligatoire**
- **Catalogue global** : Activé sur tous les contrôleurs de domaine
- **Recyclage AD** : Activé (conservation 180 jours)
- **Chiffrement** : Kerberos AES256, LDAPS obligatoire
- **Audit** : Audit complet activé sur tous les objets critiques

## Services critiques AD
```powershell
# Vérification des services AD essentiels
Get-Service | Where-Object {$_.Name -in @('ADWS','DNS','KDC','Netlogon','NTDS','W32Time')} | Format-Table Name,Status,StartType
```

Services requis :
- **ADWS** (Active Directory Web Services)
- **DNS** (Serveur DNS)
- **KDC** (Kerberos Key Distribution Center)
- **Netlogon** (Service d'ouverture de session réseau)
- **NTDS** (Active Directory Domain Services)
- **W32Time** (Service de temps Windows)

# 3. CONFIGURATION DNS

## Zones DNS principales
```powershell
# Zone de recherche directe
Add-DnsServerPrimaryZone -Name "hn.gua.local" -ZoneFile "hn.gua.local.dns"

# Zone de recherche inversée
Add-DnsServerPrimaryZone -NetworkID "192.168.10.0/24" -ZoneFile "10.168.192.in-addr.arpa.dns"
```

## Enregistrements DNS critiques
```
# Enregistrements SRV requis pour AD
_ldap._tcp.hn.gua.local.        SRV 0 100 389 dc1.hn.gua.local.
_kerberos._tcp.hn.gua.local.    SRV 0 100 88  dc1.hn.gua.local.
_gc._tcp.hn.gua.local.          SRV 0 100 3268 dc1.hn.gua.local.
_kpasswd._tcp.hn.gua.local.     SRV 0 100 464 dc1.hn.gua.local.

# Enregistrements A
dc1.hn.gua.local.               A   192.168.10.1
hn.gua.local.                   A   192.168.10.1

# Enregistrement PTR
1.10.168.192.in-addr.arpa.      PTR dc1.hn.gua.local.
```

## Configuration DNS avancée (Production)
```powershell
# Activation de la mise à jour dynamique sécurisée UNIQUEMENT
Set-DnsServerPrimaryZone -Name "hn.gua.local" -DynamicUpdate "Secure"

# Configuration des redirecteurs DNS (multiples pour redondance)
Add-DnsServerForwarder -IPAddress "1.1.1.1","1.0.0.1","8.8.8.8","8.8.4.4"

# Activation du nettoyage automatique des enregistrements obsolètes
Set-DnsServerScavenging -ScavengingState $true -ScavengingInterval 7.00:00:00
Set-DnsServerZoneAging -Name "hn.gua.local" -Aging $true -RefreshInterval 7.00:00:00 -NoRefreshInterval 7.00:00:00

# Configuration DNS sécurisée
Set-DnsServerResponseRateLimiting -Mode Enable -ResponsesPerSec 5 -ErrorsPerSec 5
Set-DnsServerCache -MaxKBSize 0 -MaxNegativeTtl 00:15:00 -MaxTtl 1.00:00:00

# Activation du DNS over HTTPS (DoH) si supporté
Set-DnsServerDoh -ServerCertificate $cert -Enable $true

# Configuration de la journalisation DNS détaillée
Set-DnsServerDiagnostics -All $true
```

## Vérification DNS
```powershell
# Test de résolution DNS
nslookup hn.gua.local 192.168.10.1
nslookup dc1.hn.gua.local 192.168.10.1

# Vérification des enregistrements SRV
nslookup -type=SRV _ldap._tcp.hn.gua.local 192.168.10.1
```

# 4. CONFIGURATION GROUP POLICY OBJECTS (GPO)

## Structure GPO recommandée (Production)
```
Group Policy Objects
├── Default Domain Policy (modifiée avec soin)
├── Default Domain Controllers Policy (durcie)
└── Production GPOs
    ├── Security Baseline
    │   ├── GPO_Security_Baseline_Servers
    │   ├── GPO_Security_Baseline_Workstations
    │   ├── GPO_Security_Baseline_Executive
    │   └── GPO_Compliance_SOX_HIPAA
    ├── Computer Configuration
    │   ├── GPO_Windows_Updates_Critical
    │   ├── GPO_Windows_Updates_Standard
    │   ├── GPO_Firewall_Rules_Servers
    │   ├── GPO_Firewall_Rules_Workstations
    │   ├── GPO_BitLocker_Encryption
    │   ├── GPO_LAPS_Configuration
    │   ├── GPO_PowerShell_Restrictions
    │   └── GPO_Software_Installation
    └── User Configuration
        ├── GPO_Desktop_Settings_Standard
        ├── GPO_Desktop_Settings_Executive
        ├── GPO_Drive_Mapping_Departmental
        ├── GPO_Password_Policy_Complex
        ├── GPO_Application_Restrictions_Standard
        ├── GPO_Application_Restrictions_High_Security
        ├── GPO_USB_Device_Control
        └── GPO_Screen_Lock_Policy
```

## GPO de sécurité essentielles
```powershell
# Création d'une GPO de sécurité
New-GPO -Name "GPO_Security_Baseline" -Comment "Baseline de sécurité pour le domaine"

# Liaison à l'OU
New-GPLink -Name "GPO_Security_Baseline" -Target "OU=Workstations,DC=hn,DC=gua,DC=local"
```

## Politique de mot de passe renforcée (Production)
```
Configuration ordinateur > Stratégies > Paramètres Windows > Paramètres de sécurité > Stratégies de compte > Stratégie de mot de passe

- Appliquer l'historique des mots de passe : 8 mots de passe mémorisés
- Âge maximal du mot de passe : 90 jours 
- Longueur minimale du mot de passe : 14 caractères (comptes privilégiés : 16)
- Le mot de passe doit respecter des exigences de complexité : Activé
- Stocker les mots de passe en utilisant un chiffrement réversible : Désactivé
```
![Capture d'écran 2025-04-13 134430](https://github.com/user-attachments/assets/1d06c334-719b-40f6-ae38-0bdd3109ae90)

![Capture d'écran 2025-04-11 171220](https://github.com/user-attachments/assets/6ac4714c-7ef1-4552-b194-dfbb9e65512f)

![Capture d'écran 2025-04-11 171138](https://github.com/user-attachments/assets/cb77f705-0fa4-4839-b7e2-6079d81e1f9c)

![Capture d'écran 2025-04-11 171039](https://github.com/user-attachments/assets/97ee1adf-1a18-4c82-91d3-c95b6787a163)

![Capture d'écran 2025-04-11 171007](https://github.com/user-attachments/assets/3ae5cd0e-822d-46c8-962c-6c2408308ad5)

![Capture d'écran 2025-04-11 170926](https://github.com/user-attachments/assets/ccff46b6-a711-47b5-9ac1-184c4297e7ed)

![Capture d'écran 2025-04-11 170900](https://github.com/user-attachments/assets/864a7152-d439-46ea-a951-49e9d4164279)

![Capture d'écran 2025-04-11 170211](https://github.com/user-attachments/assets/c5560397-6fe0-444e-b4ee-7fd541b98c97)

## Politique de verrouillage de compte renforcée
```
Configuration ordinateur > Stratégies > Paramètres Windows > Paramètres de sécurité > Stratégies de compte > Stratégie de verrouillage du compte

- Seuil de verrouillage du compte : 3 tentatives non valides (comptes privilégiés : 2)
- Durée de verrouillage du compte : 60 minutes (ou déverrouillage manuel)
- Remettre le compteur de verrouillages du compte à zéro après : 60 minutes
```

## GPO de sécurité avancée (Compliance)
```powershell
# Création d'une GPO de sécurité renforcée
New-GPO -Name "GPO_Security_Baseline_Production" -Comment "Baseline de sécurité production - Compliance SOX/HIPAA"

# Configuration avancée de sécurité
Configuration ordinateur > Stratégies > Paramètres Windows > Paramètres de sécurité > Stratégies locales > Stratégie d'audit

Audit des événements critiques :
- Audit de l'ouverture de session du compte : Réussite et Échec
- Audit de la gestion des comptes : Réussite et Échec  
- Audit de l'accès aux objets : Réussite et Échec
- Audit des modifications de stratégie : Réussite et Échec
- Audit de l'utilisation des privilèges : Réussite et Échec
- Audit du suivi des processus : Réussite uniquement
- Audit des événements système : Réussite et Échec
```

## GPO de mappage de lecteurs réseau
```powershell
# Création de la GPO de mappage de lecteurs
New-GPO -Name "GPO_Drive_Mapping" -Comment "Mappage automatique des lecteurs réseau"

# Configuration via GPMC (exemple de paramètres)
Configuration utilisateur > Préférences > Paramètres Windows > Mappages de lecteurs
- Lecteur H: \\dc1.hn.gua.local\home$\%username%
- Lecteur P: \\dc1.hn.gua.local\public$
- Lecteur S: \\dc1.hn.gua.local\shared$
```

## GPO de fond d'écran
```
Configuration utilisateur > Stratégies > Modèles d'administration > Bureau > Bureau
- Papier peint du Bureau : \\dc1.hn.gua.local\sysvol\hn.gua.local\wallpaper\corporate_wallpaper.jpg
- Style du papier peint du Bureau : Centré/Étendu/Ajusté
```

## GPO de restriction d'applications
```
Configuration utilisateur > Stratégies > Paramètres Windows > Paramètres de sécurité > Stratégies de restriction logicielle

Règles supplémentaires :
- Interdire l'exécution de cmd.exe pour les utilisateurs standards
- Bloquer l'accès au Gestionnaire des tâches
- Restreindre l'accès au Panneau de configuration
```

## Vérification et maintenance des GPO
```powershell
# Génération du rapport GPO
Get-GPOReport -All -ReportType HTML -Path "C:\GPO_Reports\All_GPOs_Report.html"

# Vérification des liens GPO
Get-GPInheritance -Target "OU=Workstations,DC=hn,DC=gua,DC=local"

# Forcer la mise à jour des GPO sur le serveur
Invoke-GPUpdate -Computer "dc1.hn.gua.local" -Force
```

# 5. CONFIGURATION PARE-FEU WINDOWS (PRODUCTION)

## Configuration Windows Defender Firewall
```powershell
# Activation du pare-feu sur tous les profils
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled True

# Configuration stricte par défaut
Set-NetFirewallProfile -Profile Domain,Public,Private -DefaultInboundAction Block
Set-NetFirewallProfile -Profile Domain,Public,Private -DefaultOutboundAction Allow

# Journalisation activée
Set-NetFirewallProfile -Profile Domain,Public,Private -LogAllowed True -LogBlocked True -LogMaxSizeKilobytes 32767
```

## Règles de pare-feu pour Active Directory (Production)
```powershell
# Ports essentiels pour AD DS - Règles entrantes
New-NetFirewallRule -DisplayName "AD-DNS-TCP-In" -Direction Inbound -Protocol TCP -LocalPort 53 -Action Allow -Profile Domain
New-NetFirewallRule -DisplayName "AD-DNS-UDP-In" -Direction Inbound -Protocol UDP -LocalPort 53 -Action Allow -Profile Domain
New-NetFirewallRule -DisplayName "AD-Kerberos-TCP-In" -Direction Inbound -Protocol TCP -LocalPort 88 -Action Allow -Profile Domain
New-NetFirewallRule -DisplayName "AD-Kerberos-UDP-In" -Direction Inbound -Protocol UDP -LocalPort 88 -Action Allow -Profile Domain
New-NetFirewallRule -DisplayName "AD-RPC-Endpoint-In" -Direction Inbound -Protocol TCP -LocalPort 135 -Action Allow -Profile Domain
New-NetFirewallRule -DisplayName "AD-NetBIOS-In" -Direction Inbound -Protocol TCP -LocalPort 139 -Action Allow -Profile Domain
New-NetFirewallRule -DisplayName "AD-LDAP-TCP-In" -Direction Inbound -Protocol TCP -LocalPort 389 -Action Allow -Profile Domain
New-NetFirewallRule -DisplayName "AD-LDAP-UDP-In" -Direction Inbound -Protocol UDP -LocalPort 389 -Action Allow -Profile Domain
New-NetFirewallRule -DisplayName "AD-SMB-In" -Direction Inbound -Protocol TCP -LocalPort 445 -Action Allow -Profile Domain
New-NetFirewallRule -DisplayName "AD-Kerberos-Pwd-TCP-In" -Direction Inbound -Protocol TCP -LocalPort 464 -Action Allow -Profile Domain
New-NetFirewallRule -DisplayName "AD-Kerberos-Pwd-UDP-In" -Direction Inbound -Protocol UDP -LocalPort 464 -Action Allow -Profile Domain
New-NetFirewallRule -DisplayName "AD-LDAPS-In" -Direction Inbound -Protocol TCP -LocalPort 636 -Action Allow -Profile Domain
New-NetFirewallRule -DisplayName "AD-Global-Catalog-In" -Direction Inbound -Protocol TCP -LocalPort 3268 -Action Allow -Profile Domain
New-NetFirewallRule -DisplayName "AD-Global-Catalog-SSL-In" -Direction Inbound -Protocol TCP -LocalPort 3269 -Action Allow -Profile Domain

# Plage RPC dynamique (Windows Server 2019+)
New-NetFirewallRule -DisplayName "AD-RPC-Dynamic-In" -Direction Inbound -Protocol TCP -LocalPort 49152-65535 -Action Allow -Profile Domain

# Ports pour l'administration sécurisée
New-NetFirewallRule -DisplayName "RDP-Admin-In" -Direction Inbound -Protocol TCP -LocalPort 3389 -Action Allow -Profile Domain -RemoteAddress 192.168.100.0/24
New-NetFirewallRule -DisplayName "WinRM-HTTPS-In" -Direction Inbound -Protocol TCP -LocalPort 5986 -Action Allow -Profile Domain -RemoteAddress 192.168.100.0/24

# Restriction par adresse source (sécurité renforcée)
Set-NetFirewallRule -DisplayName "AD-LDAP-TCP-In" -RemoteAddress 192.168.10.0/24,192.168.20.0/24
Set-NetFirewallRule -DisplayName "AD-SMB-In" -RemoteAddress 192.168.10.0/24,192.168.20.0/24
```

## Règles sortantes restrictives (Production)
```powershell
# Blocage par défaut des connexions sortantes non autorisées
Set-NetFirewallProfile -Profile Domain -DefaultOutboundAction Block

# Autorisation des services essentiels sortants
New-NetFirewallRule -DisplayName "DNS-Out" -Direction Outbound -Protocol UDP -LocalPort Any -RemotePort 53 -Action Allow -Profile Domain
New-NetFirewallRule -DisplayName "HTTP-Out" -Direction Outbound -Protocol TCP -LocalPort Any -RemotePort 80 -Action Allow -Profile Domain
New-NetFirewallRule -DisplayName "HTTPS-Out" -Direction Outbound -Protocol TCP -LocalPort Any -RemotePort 443 -Action Allow -Profile Domain
New-NetFirewallRule -DisplayName "NTP-Out" -Direction Outbound -Protocol UDP -LocalPort Any -RemotePort 123 -Action Allow -Profile Domain
New-NetFirewallRule -DisplayName "LDAP-Replication-Out" -Direction Outbound -Protocol TCP -LocalPort Any -RemotePort 389 -RemoteAddress 192.168.10.0/24 -Action Allow -Profile Domain
New-NetFirewallRule -DisplayName "Kerberos-Out" -Direction Outbound -Protocol TCP -LocalPort Any -RemotePort 88 -RemoteAddress 192.168.10.0/24 -Action Allow -Profile Domain

# Blocage explicite des réseaux non autorisés
New-NetFirewallRule -DisplayName "Block-Internet-Outbound" -Direction Outbound -RemoteAddress Internet -Action Block -Profile Domain
```

## Configuration avancée de sécurité
```powershell
# Activation de la protection contre les attaques par déni de service
netsh advfirewall set global statefulftp disable
netsh advfirewall set global statefulpptp disable

# Configuration du logging avancé
auditpol /set /subcategory:"Filtering Platform Connection" /success:enable /failure:enable
auditpol /set /subcategory:"Filtering Platform Packet Drop" /success:disable /failure:enable
```

# 6. MONITORING ET MAINTENANCE

## Scripts de surveillance automatisée (Production)
```powershell
# Script de surveillance complet AD (à exécuter toutes les heures)
$HealthCheck = @"
# Vérification complète de la santé AD
dcdiag /v /c /d /e /s:dc1.hn.gua.local /f:C:\Logs\DCDiag_DC1.log
dcdiag /v /c /d /e /s:dc2.hn.gua.local /f:C:\Logs\DCDiag_DC2.log

# Vérification de la réplication AD (critique)
repadmin /replsummary /bysrc /bydest /sort:delta
repadmin /showrepl dc1.hn.gua.local /csv > C:\Logs\Replication_DC1.csv
repadmin /showrepl dc2.hn.gua.local /csv > C:\Logs\Replication_DC2.csv

# Test des services Kerberos et authentification
nltest /dsgetdc:hn.gua.local /force
nltest /sc_query:hn.gua.local

# Vérification des rôles FSMO
netdom query fsmo

# Test de connectivité DNS
nslookup -type=SRV _ldap._tcp.hn.gua.local
nslookup -type=SRV _kerberos._tcp.hn.gua.local
nslookup -type=SRV _gc._tcp.hn.gua.local

# Surveillance des performances critiques
Get-Counter "\DirectoryServices(NTDS)\LDAP Searches/sec"
Get-Counter "\DirectoryServices(NTDS)\LDAP Successful Binds/sec"
Get-Counter "\Memory\Available MBytes"
Get-Counter "\Processor(_Total)\% Processor Time"
"@

# Planification du script de surveillance
Register-ScheduledTask -TaskName "AD-Health-Monitor" -Trigger (New-ScheduledTaskTrigger -Once -At (Get-Date) -RepetitionInterval (New-TimeSpan -Hours 1)) -Action (New-ScheduledTaskAction -Execute "PowerShell.exe" -Argument "-File C:\Scripts\AD-Health-Monitor.ps1")
```

## Sauvegarde et restauration (Production)
```powershell
# Sauvegarde système complète quotidienne (obligatoire)
# Configuration Windows Server Backup pour sauvegarde automatique

# Sauvegarde de l'état système (inclut AD, SYSVOL, Registre)
wbadmin start systemstatebackup -backupTarget:E:\Backups\SystemState -quiet

# Sauvegarde spécifique de la base AD (NTDS)
ntdsutil "activate instance ntds" "snapshot" "create" "quit" "quit"

# Script de sauvegarde automatisée complet
$BackupScript = @"
# Sauvegarde AD complète avec rotation
param(
    [string]`$BackupPath = "E:\Backups\AD",
    [int]`$RetentionDays = 30
)

# Création du dossier de sauvegarde avec timestamp
`$Date = Get-Date -Format "yyyyMMdd_HHmmss"
`$BackupFolder = Join-Path `$BackupPath `$Date
New-Item -Path `$BackupFolder -ItemType Directory -Force

# Sauvegarde de l'état système
wbadmin start systemstatebackup -backupTarget:`$BackupFolder -quiet

# Sauvegarde des GPO
Backup-GPO -All -Path `$BackupFolder\GPO_Backup

# Export de la configuration DNS
Export-DnsServerZone -Name "hn.gua.local" -FileName "`$BackupFolder\DNS_hn.gua.local.txt"

# Nettoyage des anciennes sauvegardes
Get-ChildItem `$BackupPath | Where-Object {`$_.CreationTime -lt (Get-Date).AddDays(-`$RetentionDays)} | Remove-Item -Recurse -Force

# Vérification de l'intégrité de la sauvegarde
Test-Path "`$BackupFolder\WindowsImageBackup"
"@

# Planification de la sauvegarde quotidienne à 2h00
Register-ScheduledTask -TaskName "AD-Daily-Backup" -Trigger (New-ScheduledTaskTrigger -Daily -At "02:00") -Action (New-ScheduledTaskAction -Execute "PowerShell.exe" -Argument "-File C:\Scripts\AD-Backup.ps1")

# Test de restauration mensuelle (procédure critique)
# Documentation de procédure de restauration complète disponible dans le plan de reprise d'activité
```

## Surveillance des journaux (Production)
```powershell
# Configuration de la collecte centralisée des logs (SIEM)
# Integration avec Windows Event Forwarding (WEF)

# Journaux AD critiques à surveiller 24/7
$CriticalLogs = @(
    @{LogName='Directory Service'; EventID=@(1644,2087,2088,1311,1168)}, # Erreurs critiques AD
    @{LogName='DNS Server'; EventID=@(4013,4015,4000,150)}, # Erreurs DNS critiques
    @{LogName='System'; EventID=@(1074,6005,6006,6008,6013)}, # Événements système critiques
    @{LogName='Security'; EventID=@(4740,4625,4624,4634,4648,4656,4719,4720,4726)}, # Événements de sécurité
    @{LogName='Application'; EventID=@(1000,1002)} # Erreurs d'application
)

# Script de surveillance des événements critiques
foreach ($Log in $CriticalLogs) {
    Get-WinEvent -FilterHashtable @{LogName=$Log.LogName; ID=$Log.EventID; StartTime=(Get-Date).AddHours(-1)} -ErrorAction SilentlyContinue |
    ForEach-Object {
        # Envoi d'alerte immédiate pour événements critiques
        Send-MailMessage -To "admin@hn.gua.local" -From "dc-monitor@hn.gua.local" -Subject "ALERTE AD - Événement critique détecté" -Body $_.Message -SmtpServer "mail.hn.gua.local"
    }
}

# Configuration de l'audit avancé obligatoire
auditpol /set /subcategory:"Directory Service Access" /success:enable /failure:enable
auditpol /set /subcategory:"Directory Service Changes" /success:enable /failure:enable
auditpol /set /subcategory:"Account Management" /success:enable /failure:enable
auditpol /set /subcategory:"Logon" /success:enable /failure:enable
auditpol /set /subcategory:"Account Lockout" /success:enable /failure:enable
auditpol /set /subcategory:"Policy Change" /success:enable /failure:enable
auditpol /set /subcategory:"Privilege Use" /success:enable /failure:enable
auditpol /set /subcategory:"System" /success:enable /failure:enable

# Archivage automatique des logs (compliance)
wevtutil set-log "Directory Service" /retention:true /maxsize:1073741824
wevtutil set-log "Security" /retention:true /maxsize:2147483648
```

# 7. DÉPANNAGE ET TROUBLESHOOTING

## Commandes de diagnostic
```powershell
# Test de connectivité réseau
Test-NetConnection -ComputerName dc1.hn.gua.local -Port 389
Test-NetConnection -ComputerName dc1.hn.gua.local -Port 53

# Vérification des services AD
Get-Service ADWS,DNS,KDC,Netlogon,NTDS | Format-Table Name,Status

# Test de résolution DNS
Resolve-DnsName hn.gua.local
Resolve-DnsName _ldap._tcp.hn.gua.local -Type SRV
```

## Résolution des problèmes courants
1. **Problèmes de réplication AD** : Vérifier la connectivité réseau et les ports ouverts
2. **Erreurs DNS** : Contrôler les enregistrements SRV et la configuration des redirecteurs
3. **Problèmes GPO** : Vérifier les liens GPO et forcer la mise à jour
4. **Authentification Kerberos** : Synchroniser l'heure entre le serveur et les clients

# 8. SÉCURITÉ ET CONFORMITÉ (PRODUCTION)

## Durcissement du serveur AD (Security Hardening)
```powershell
# Désactivation des services non critiques
$ServicesToDisable = @(
    'Fax', 'TapiSrv', 'Telnet', 'SimpleHelp', 'RemoteRegistry', 
    'Browser', 'MessengerService', 'PrintSpooler'
)
foreach ($Service in $ServicesToDisable) {
    if (Get-Service $Service -ErrorAction SilentlyContinue) {
        Stop-Service $Service -Force
        Set-Service $Service -StartupType Disabled
    }
}

# Configuration des paramètres de sécurité avancés
secedit /configure /db C:\Windows\security\local.sdb /cfg C:\Scripts\SecPolicy.inf

# Activation de LDAPS obligatoire (port 636)
$cert = Get-ChildItem -Path Cert:\LocalMachine\My | Where-Object {$_.Subject -like "*dc1.hn.gua.local*"}
if ($cert) {
    # Configuration LDAPS
    $regPath = "HKLM:\SYSTEM\CurrentControlSet\Services\NTDS\Parameters"
    Set-ItemProperty -Path $regPath -Name "LDAP Server Integrity" -Value 2
}

# Restriction des protocoles d'authentification
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" -Name "LmCompatibilityLevel" -Value 5
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" -Name "NoLMHash" -Value 1

# Activation du chiffrement Kerberos AES256
ksetup /setenctypeattr hn.gua.local AES256-CTS-HMAC-SHA1-96 AES128-CTS-HMAC-SHA1-96

# Configuration des stratégies de sécurité renforcées
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" -Name "RestrictAnonymous" -Value 2
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" -Name "RestrictAnonymousSAM" -Value 1
```

## Compliance et audit (SOX, HIPAA, ISO 27001)
```powershell
# Configuration de l'audit complet pour compliance réglementaire
$AuditCategories = @(
    "Directory Service Access",
    "Directory Service Changes", 
    "Directory Service Replication",
    "Account Management",
    "Account Logon",
    "Logon",
    "Object Access",
    "Policy Change",
    "Privilege Use",
    "System",
    "Global Object Access Auditing"
)

foreach ($Category in $AuditCategories) {
    auditpol /set /subcategory:"$Category" /success:enable /failure:enable
}

# Configuration de l'audit des objets AD critiques
$CriticalOUs = @(
    "OU=Domain_Admins,OU=Users_Accounts,DC=hn,DC=gua,DC=local",
    "OU=Server_Admins,OU=Users_Accounts,DC=hn,DC=gua,DC=local",
    "CN=Domain Controllers,DC=hn,DC=gua,DC=local"
)

foreach ($OU in $CriticalOUs) {
    dsacls $OU /I:T /S
}

# Mise en place de la surveillance des comptes privilégiés
$PrivilegedGroups = @("Domain Admins", "Enterprise Admins", "Schema Admins", "Administrators")
foreach ($Group in $PrivilegedGroups) {
    Get-ADGroupMember $Group | Select-Object Name,SamAccountName,LastLogonDate | 
    Export-Csv "C:\Audit\PrivilegedAccounts_$(Get-Date -Format 'yyyyMMdd').csv" -Append
}
```

## Protection contre les attaques avancées
```powershell
# Installation et configuration de Microsoft Defender pour Identity
# (Anciennement Azure ATP)

# Protection contre les attaques Golden Ticket
# Rotation régulière du compte KRBTGT (tous les 6 mois minimum)
Reset-KrbtgtKeys -Domain hn.gua.local

# Activation de Credential Guard
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\DeviceGuard" -Name "EnableVirtualizationBasedSecurity" -Value 1
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\DeviceGuard" -Name "RequirePlatformSecurityFeatures" -Value 3

# Configuration de LAPS (Local Administrator Password Solution)
Update-AdmPwdADSchema
Set-AdmPwdComputerSelfPermission -OrgUnit "OU=Workstations,DC=hn,DC=gua,DC=local"

# Protection contre DCSync et autres attaques AD
$DCPermissions = Get-Acl "AD:\DC=hn,DC=gua,DC=local"
# Restriction des permissions de réplication aux seuls comptes autorisés
```

## Gestion des certificats et PKI
```powershell
# Installation d'Active Directory Certificate Services (Production)
Install-WindowsFeature -Name ADCS-Cert-Authority -IncludeManagementTools
Install-AdcsCertificationAuthority -CAType EnterpriseRootCA -CryptoProviderName "RSA#Microsoft Software Key Storage Provider" -KeyLength 4096 -HashAlgorithmName SHA256

# Configuration des templates de certificats pour LDAPS, Kerberos, etc.
```

# 9. PERFORMANCES ET OPTIMISATION (PRODUCTION)

## Optimisation matérielle et système
```powershell
# Configuration des performances AD optimales
# Placement des fichiers de base de données sur disques séparés
$NTDSPath = "D:\Windows\NTDS"  # SSD dédié pour base de données
$LogPath = "E:\Windows\NTDS"   # SSD séparé pour logs de transaction
$SysvolPath = "F:\Windows\SYSVOL"  # Disque pour SYSVOL

# Optimisation de la mémoire virtuelle
$PageFileSize = [math]::Round((Get-CimInstance Win32_PhysicalMemory | Measure-Object Capacity -Sum).Sum / 1GB * 1.5)
Set-CimInstance -Query "SELECT * FROM Win32_PageFileSetting WHERE Name='C:\\pagefile.sys'" -Property @{InitialSize=$PageFileSize*1024; MaximumSize=$PageFileSize*1024}

# Configuration des paramètres de performance AD
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\NTDS\Parameters" -Name "Database reclaim working space (percent)" -Value 10
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\NTDS\Parameters" -Name "Replicator notify pause after modify (secs)" -Value 5

# Optimisation DNS pour les performances
Set-DnsServerCache -MaxKBSize 0 -MaxNegativeTtl 00:15:00 -MaxTtl 1.00:00:00
Set-DnsServerResponseRateLimiting -Mode Enable -ResponsesPerSec 10 -ErrorsPerSec 5
```

## Surveillance des performances critiques
```powershell
# Script de monitoring des performances AD (exécution continue)
$PerformanceCounters = @(
    "\DirectoryServices(NTDS)\LDAP Client Sessions",
    "\DirectoryServices(NTDS)\LDAP Active Threads", 
    "\DirectoryServices(NTDS)\LDAP Searches/sec",
    "\DirectoryServices(NTDS)\LDAP Successful Binds/sec",
    "\DirectoryServices(NTDS)\LDAP Bind Time",
    "\DirectoryServices(NTDS)\DRA Inbound Values (DNs only)/sec",
    "\DirectoryServices(NTDS)\DRA Outbound Values (DNs only)/sec",
    "\DirectoryServices(NTDS)\DS Directory Reads/sec",
    "\DirectoryServices(NTDS)\DS Directory Writes/sec",
    "\DirectoryServices(NTDS)\Database == Instances(lsass/NTDSAI:0) Cache % Hit",
    "\Memory\Available MBytes",
    "\Memory\Pool Nonpaged Bytes",
    "\Processor(_Total)\% Processor Time",
    "\PhysicalDisk(_Total)\Avg. Disk Queue Length",
    "\PhysicalDisk(_Total)\% Disk Time",
    "\Network Interface(*)\Bytes Total/sec"
)

# Collecte et analyse des métriques
$PerfData = Get-Counter -Counter $PerformanceCounters -SampleInterval 30 -MaxSamples 120

# Seuils d'alerte production
$Thresholds = @{
    "CPUUsage" = 80
    "MemoryAvailable" = 2048  # MB
    "LDAPBindTime" = 100      # ms
    "DiskQueueLength" = 2
    "LDAPSearchesPerSec" = 1000
}

# Génération d'alertes automatiques si seuils dépassés
```

## Métriques de performance cibles (Production)
- **Utilisation CPU** : < 70% en moyenne (pics < 90%)
- **Mémoire disponible** : > 2 Go en permanence  
- **Temps de réponse LDAP** : < 50ms (95e percentile)
- **Recherches LDAP/sec** : < 1000 (surveillance des pics)
- **Temps de liaison LDAP** : < 100ms
- **Latence de réplication** : < 15 minutes entre sites
- **Espace disque libre** : > 20% sur toutes les partitions
- **Queue Length disque** : < 2 en moyenne

## Optimisation de la réplication AD
```powershell
# Configuration de la topologie de réplication optimisée
# Création de sites AD pour optimiser le trafic réseau
New-ADReplicationSite -Name "Site-Principal" -Description "Site principal datacenter"
New-ADReplicationSite -Name "Site-Secondaire" -Description "Site secondaire DR"

# Configuration des liens de sites avec bande passante appropriée
New-ADReplicationSiteLink -Name "Principal-Secondaire" -SitesIncluded "Site-Principal","Site-Secondaire" -Cost 100 -ReplicationFrequencyInMinutes 15

# Optimisation des intervalles de réplication
Set-ADReplicationSiteLink -Identity "Principal-Secondaire" -ReplicationFrequencyInMinutes 15 -Options 1

# Configuration du Bridgehead Server préféré
Set-ADReplicationConnection -Identity "ConnexionReplication" -ReplicateFromDirectoryServer "dc1.hn.gua.local"
```

## Maintenance préventive
```powershell
# Script de maintenance hebdomadaire AD
$MaintenanceScript = @"
# Compactage en ligne de la base de données AD
ntdsutil "activate instance ntds" "files" "compact to C:\Temp\NTDS_Compact" "quit" "quit"

# Défragmentation des fichiers de base de données
esentutl /d "D:\Windows\NTDS\ntds.dit" /t "C:\Temp\ntds_temp.dit"

# Nettoyage des métadonnées obsolètes
repadmin /removelingeringobjects dc1.hn.gua.local dc2.hn.gua.local "DC=hn,DC=gua,DC=local"

# Vérification et réparation de la base de données
ntdsutil "activate instance ntds" "files" "integrity" "quit" "quit"

# Nettoyage des journaux de transaction
ntdsutil "activate instance ntds" "files" "delete logs" "quit" "quit"

# Optimisation des index AD
ldifde -f C:\Temp\AD_Export.ldif -d "DC=hn,DC=gua,DC=local"
"@

# Planification de la maintenance le dimanche à 3h00
Register-ScheduledTask -TaskName "AD-Weekly-Maintenance" -Trigger (New-ScheduledTaskTrigger -Weekly -WeeksInterval 1 -DaysOfWeek Sunday -At "03:00") -Action (New-ScheduledTaskAction -Execute "PowerShell.exe" -Argument "-File C:\Scripts\AD-Maintenance.ps1")
```

# 10. PLAN DE CONTINUITÉ D'ACTIVITÉ (PCA)

## Stratégie de haute disponibilité
```powershell
# Configuration du clustering Windows Server (si applicable)
# Installation du rôle Failover Clustering
Install-WindowsFeature -Name Failover-Clustering -IncludeManagementTools

# Mise en place de la réplication SYSVOL via DFSR
dfsrmig /setglobalstate 2  # Migration vers DFSR
dfsrmig /setglobalstate 3  # Finalisation de la migration

# Configuration de la surveillance de santé automatique
$HAScript = @"
# Script de surveillance haute disponibilité
param()

# Vérification de l'état des contrôleurs de domaine
$DCs = @("dc1.hn.gua.local", "dc2.hn.gua.local")
$FailedDCs = @()

foreach ($DC in $DCs) {
    $TestResult = Test-NetConnection -ComputerName $DC -Port 389 -WarningAction SilentlyContinue
    if (-not $TestResult.TcpTestSucceeded) {
        $FailedDCs += $DC
        # Déclenchement des procédures de basculement automatique
        Write-EventLog -LogName Application -Source "AD-HA-Monitor" -EventId 1001 -EntryType Error -Message "Contrôleur de domaine $DC indisponible"
    }
}

# Basculement automatique des services DNS si nécessaire
if ($FailedDCs.Count -gt 0) {
    # Mise à jour des enregistrements DNS pour rediriger le trafic
    # Notification automatique de l'équipe d'administration
}
"@
```

## Procédures de sauvegarde stratégiques
```powershell
# Sauvegarde multi-sites avec réplication géographique
$BackupStrategy = @"
# Stratégie de sauvegarde 3-2-1 (3 copies, 2 supports, 1 externe)

# Sauvegarde locale quotidienne (Tier 1)
wbadmin start systemstatebackup -backupTarget:E:\Backups\Local -quiet

# Sauvegarde sur site distant (Tier 2) - via robocopy chiffré
robocopy E:\Backups\Local \\backup-server.remote.local\AD_Backups /E /R:3 /W:30 /LOG:C:\Logs\Remote_Backup.log

# Sauvegarde cloud sécurisée (Tier 3) - Azure Backup ou équivalent
Start-OBBackup -Policy (Get-OBPolicy)

# Test de restauration automatisé mensuel
$TestRestore = {
    # Création d'un environnement de test isolé
    # Restauration complète pour vérification d'intégrité
    # Validation des services AD, DNS, GPO
    # Génération du rapport de test de restauration
}
"@
```

## Plan de reprise d'activité (PRA)
```powershell
# Procédures de récupération d'urgence (RTO: 4h, RPO: 1h)

# Scénario 1: Panne du contrôleur principal (DC1)
$DC1FailoverProcedure = @"
1. Vérification automatique de l'indisponibilité de DC1
2. Promotion automatique de DC2 comme PDC Emulator
3. Transfert des rôles FSMO vers DC2
4. Mise à jour des enregistrements DNS
5. Notification des utilisateurs et services
6. Démarrage de la reconstruction de DC1
"@

# Script de transfert automatique des rôles FSMO
Move-ADDirectoryServerOperationMasterRole -Identity "dc2.hn.gua.local" -OperationMasterRole SchemaMaster,DomainNamingMaster,RIDMaster -Force

# Scénario 2: Corruption de la base de données AD
$DatabaseRecoveryProcedure = @"
1. Arrêt des services AD sur tous les DCs
2. Restauration de la dernière sauvegarde système valide
3. Redémarrage en mode restauration AD (DSRM)  
4. Restauration autoritaire si nécessaire
5. Vérification de l'intégrité de la base de données
6. Redémarrage des services et réplication
"@
```

# 11. PROCÉDURES D'INCIDENT ET ESCALADE

## Classification des incidents
```
CRITIQUE (P1) - RTO: 1h
- Panne complète des services d'authentification
- Corruption de la base de données AD
- Compromission de sécurité majeure

MAJEUR (P2) - RTO: 4h  
- Panne d'un contrôleur de domaine
- Problèmes de réplication AD étendus
- Dysfonctionnements DNS critiques

MINEUR (P3) - RTO: 8h
- Problèmes de GPO localisés
- Problèmes de performance
- Alertes de surveillance

INFORMATIF (P4) - RTO: 24h
- Maintenance préventive
- Optimisations de performance
- Mises à jour non critiques
```

## Contacts d'escalade
```
Niveau 1: Équipe Support (24/7)
- Email: support@hn.gua.local
- Téléphone: +33 1 XX XX XX XX

Niveau 2: Administrateurs Système Senior
- Email: sysadmin@hn.gua.local  
- Téléphone: +33 6 XX XX XX XX (garde)

Niveau 3: Architecte Infrastructure
- Email: architect@hn.gua.local
- Téléphone: +33 6 XX XX XX XX (urgence uniquement)

Direction IT (incidents critiques uniquement)
- Email: itdirector@hn.gua.local
- Téléphone: +33 6 XX XX XX XX
```

# 12. DOCUMENTATION COMPLÉMENTAIRE

## Documents de référence requis
- Plan de reprise d'activité (PRA) détaillé
- Procédures de sauvegarde et restauration complètes  
- Guide de durcissement sécuritaire AD
- Matrice des permissions et accès
- Plan de tests de sécurité (pentest)
- Documentation de conformité réglementaire
- Procédures de gestion des correctifs
- Guide d'administration quotidienne

## Formation et certification requises
- Certification Microsoft AD/Azure AD pour les administrateurs
- Formation sécurité IT (ISO 27001, CISSP)
- Habilitation sécuritaire selon le niveau de classification
- Formation continue sur les menaces cyber

---

**ENVIRONNEMENT DE PRODUCTION** : Cette fiche technique est conçue pour un déploiement en production avec toutes les exigences de sécurité, haute disponibilité et performance requises.
