# Guide d'implémentation d'un système de déploiement Windows Master/MDT

## Sommaire

| Section | Description |
|---------|-------------|
| [Introduction](#introduction) | Contexte, objectifs et contraintes |
| [Architecture technique](#architecture-technique) | Vue d'ensemble et composants |
| [Configuration du Master Windows](#configuration-du-master-windows) | Préparation et installation |
| [Création et capture d'une image Master](#création-et-capture-dune-image-master) | Processus détaillé de création d'image |
| [Configuration du serveur MDT](#configuration-du-serveur-mdt) | Installation et paramétrage |
| [Préparation du partage de déploiement](#préparation-du-partage-de-déploiement) | Configuration complète du partage MDT |
| [Processus de maintenance](#processus-de-maintenance) | Cycle de mise à jour et documentation |
| [Déploiement des images avec intégration AD](#déploiement-des-images-avec-intégration-ad) | Processus et configuration |
| [Intégration Active Directory](#intégration-active-directory) | Configuration détaillée |
| [Cas pratiques de déploiement](#cas-pratiques-de-déploiement) | Solutions et problèmes courants |
| [Considérations de sécurité](#considérations-de-sécurité) | Protection et conformité |

## Introduction

Ce document présente la mise en œuvre d'une solution de déploiement d'images Windows basée sur l'approche Master/MDT (Microsoft Deployment Toolkit) pour standardiser les postes de travail et optimiser leur déploiement au sein de l'entreprise Oodrive.

### Contexte actuel

L'entreprise fait face à plusieurs problématiques liées au déploiement manuel des postes de travail :
- Temps d'installation excessif par poste
- Hétérogénéité des configurations
- Maintenance et mises à jour complexes
- Gestion inefficace des logiciels et configurations spécifiques

### Objectifs

- Standardiser l'environnement de travail
- Réduire le temps de déploiement
- Simplifier la maintenance et les mises à jour
- Garantir la sécurité et la conformité 
- Faciliter l'intégration des nouveaux collaborateurs

### Contraintes

- Respect des normes de sécurité de l'entreprise
- Compatibilité avec l'infrastructure existante
- Nécessité de mises à jour régulières
- Intégration avec l'infrastructure Active Directory existante

## Architecture technique

### Vue d'ensemble

L'infrastructure repose sur une architecture Master/MDT permettant de créer et déployer des images standardisées sur les postes de travail via un serveur centralisé.

### Composants principaux

#### Serveur Master (Proxmox)
- **Rôle** : Hébergement de la VM Master Windows
- **Configuration** :
  - Système : Proxmox VE
  - Ressources VM Master : 4 cœurs CPU, 8 Go RAM, 100 Go stockage

#### Machine virtuelle Master
- **Système** : Windows 10/11 Professionnel
- **Configuration** :
  - Certificats WiFi liés à l'Active Directory
  - Firefox comme navigateur par défaut
  - Suite Microsoft Office complète
  - Logiciels standards de l'entreprise

#### Serveur de déploiement MDT
- **Système** : Windows Server avec rôles MDT et WDS
- **Fonctions** :
  - Réception des images Master après mise à jour
  - Gestion du déploiement vers les postes clients
  - Configuration automatique selon les règles définies
  - Intégration des postes à l'Active Directory

#### Outils principaux
- **Chocolatey (Choco)** : Gestionnaire de paquets Windows pour l'installation et les mises à jour
- **WinSCP** : Outil de transfert sécurisé pour les images Master
- **MDT** : Solution de déploiement des images
- **PXE** : Environnement d'exécution de préamorçage pour le déploiement réseau

## Configuration du Master Windows

### Préparation de l'environnement

1. **Installation de la VM Master sous Proxmox**
   - Création de la VM avec allocation des ressources requises
   - Installation de Windows 10/11 Professionnel

2. **Configuration initiale du système**
   - Paramétrage du réseau
   - Installation des pilotes spécifiques
   - Application des mises à jour Windows

### Installation des logiciels standards

1. **Mise en place de Chocolatey**

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force
iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
```

2. **Installation des logiciels via Chocolatey**

```powershell
choco install firefox office365proplus adobereader 7zip -y
```

3. **Installation des utilitaires spécifiques**
   - Intégration des outils internes
   - Déploiement des certificats WiFi
   - Configuration des politiques de sécurité

### Configurations spécifiques
- **Paramètres réseau** pour l'intégration Active Directory
- **Sécurité** selon les politiques de l'entreprise
- **Configuration utilisateur par défaut**
- **Personnalisation de l'interface**

## Création et capture d'une image Master

### Préparation de l'image pour la capture

1. **Nettoyage du système**
   - Suppression des fichiers temporaires et inutiles
   - Désactivation des services non essentiels pour la capture

```powershell
# Nettoyage avant capture
Stop-Service -Name wuauserv
Remove-Item -Path C:\Windows\SoftwareDistribution\* -Recurse -Force -ErrorAction SilentlyContinue
Remove-Item -Path C:\Windows\Temp\* -Recurse -Force -ErrorAction SilentlyContinue
Clear-RecycleBin -Force -ErrorAction SilentlyContinue
```

2. **Optimisation du système**
   - Défragmentation du disque
   - Nettoyage des composants Windows inutilisés

```powershell
# Optimisation du disque
Optimize-Volume -DriveLetter C -Defrag
# Nettoyage des composants Windows
DISM /Online /Cleanup-Image /StartComponentCleanup
```

### Processus de capture de l'image

1. **Exécution de Sysprep**
   - Préparation du système pour le déploiement
   - Généralisation des paramètres spécifiques à la machine

```powershell
# Exécution du processus Sysprep
Start-Process -FilePath "C:\Windows\System32\Sysprep\sysprep.exe" -ArgumentList "/generalize /oobe /shutdown" -NoNewWindow -Wait
```

2. **Capture via MDT**
   - Configuration d'une séquence de tâches de capture dans MDT
   - Création du partage de capture sur le serveur MDT

```powershell
# Configuration de la séquence de capture (à exécuter sur le serveur MDT)
New-Item -Path "DS:\Task Sequences" -Enable "True" -Name "Capture Master" -Comments "Séquence de capture d'image Master" -Template "Client.xml"
```

3. **Démarrage de la machine Master en PXE**
   - Sélection de la séquence de capture dans l'environnement MDT
   - Configuration des paramètres de capture (chemin de stockage, nom de l'image)

4. **Création automatique du fichier WIM**
   - Processus de capture automatisé par MDT
   - Stockage de l'image au format WIM dans le partage de déploiement

```
# Le processus génère un fichier WIM qui sera stocké dans le répertoire:
# \\serveur\DeploymentShare$\Captures\Master_YYYY-MM-DD.wim
```

5. **Importation de l'image capturée dans MDT**

```powershell
# Importation de l'image capturée dans le partage de déploiement
Import-MDTOperatingSystem -Path "DS:\Operating Systems" -SourceFile "\\serveur\DeploymentShare$\Captures\Master_YYYY-MM-DD.wim" -DestinationFolder "Windows10-Master-YYYY-MM-DD"
```

## Configuration du serveur MDT

### Installation du serveur MDT

1. **Prérequis**
   - Windows Server avec interface graphique
   - Accès réseau aux postes clients
   - Droits d'administration sur le domaine

2. **Installation des composants**
   - Installation de Windows ADK (Assessment and Deployment Kit)
   - Installation de MDT (Microsoft Deployment Toolkit)
   - Configuration du rôle WDS (Windows Deployment Services)

3. **Configuration de la base MDT**

```powershell
New-Item -Path "C:\DeploymentShare" -ItemType Directory
Import-Module "C:\Program Files\Microsoft Deployment Toolkit\bin\MicrosoftDeploymentToolkit.psd1"
New-PSDrive -Name "DS" -PSProvider MDTProvider -Root "C:\DeploymentShare"
```

## Préparation du partage de déploiement

### Création et configuration du partage

1. **Établissement de la structure du partage**

```powershell
# Création du partage réseau
New-Item -Path "D:\DeploymentShare" -ItemType Directory -Force
New-SmbShare -Name "DeploymentShare$" -Path "D:\DeploymentShare" -FullAccess "Administrators"

# Initialisation du partage MDT
New-PSDrive -Name "DS" -PSProvider MDTProvider -Root "D:\DeploymentShare"
New-Item -Path "DS:\Boot" -Enable "True" -Name "GenericBoot" -Comments "Images d'amorçage PXE"
```

2. **Configuration des dossiers principaux**
   - Applications: stockage des applications à déployer
   - Operating Systems: stockage des images OS capturées
   - Out-of-Box Drivers: organisation des pilotes par fabricant/modèle
   - Task Sequences: définition des séquences de déploiement
   - Media: support pour création de médias bootables

3. **Paramétrage des propriétés du partage**

```powershell
# Configuration des propriétés du partage de déploiement
Set-ItemProperty DS:\ -Name "UNCPath" -Value "\\serveur\DeploymentShare$"
Set-ItemProperty DS:\ -Name "EnableMulticast" -Value "True"
```

### Configuration des règles de déploiement

1. **Création du fichier CustomSettings.ini**
   - Configuration des paramètres d'automatisation
   - Intégration Active Directory automatique

```ini
[Settings]
Priority=Default
Properties=MyCustomProperty

[Default]
OSInstall=Y
SkipCapture=YES
SkipAdminPassword=YES
SkipProductKey=YES
SkipComputerBackup=YES
SkipBitLocker=YES
TimeZoneName=Romance Standard Time
JoinDomain=domaineentreprise.local
DomainAdmin=DOMAINE\AdminMDT
DomainAdminPassword=MotDePasseSécurisé
MachineObjectOU=OU=Ordinateurs,OU=Entreprise,DC=domaineentreprise,DC=local
```

2. **Paramétrage de l'intégration AD**
   - Configuration des informations d'identification pour l'intégration au domaine
   - Définition de l'unité d'organisation cible pour les ordinateurs

3. **Génération des images d'amorçage**

```powershell
# Génération des images d'amorçage pour PXE
Update-MDTDeploymentShare -Path "D:\DeploymentShare" -Force
```

4. **Configuration des fichiers de réponse**
   - Configuration des réponses automatisées pour Windows
   - Personnalisation des paramètres régionaux et linguistiques

### Importation et organisation des composants

1. **Importation des systèmes d'exploitation**

```powershell
Import-MDTOperatingSystem -Path "DS:\Operating Systems" -SourcePath "D:\Sources\Windows10" -DestinationFolder "Windows10-Original"
```

2. **Importation des applications**

```powershell
Import-MDTApplication -Path "DS:\Applications" -Name "Mozilla Firefox" -CommandLine "Firefox_Setup.exe /S" -WorkingDirectory ".\Applications\Mozilla Firefox" -ApplicationSourcePath "D:\Sources\Applications\Firefox"
```

3. **Organisation des pilotes**

```powershell
Import-MDTDriver -Path "DS:\Out-of-Box Drivers\Dell\Latitude 5520" -SourcePath "D:\Sources\Drivers\Dell\Latitude5520"
```

4. **Création des séquences de tâches**

```powershell
New-Item -Path "DS:\Task Sequences" -Enable "True" -Name "Deploy Windows 10" -Comments "Déploiement standard Windows 10" -Template "Client.xml" -OperatingSystemPath "DS:\Operating Systems\Windows 10 Enterprise x64" -FullName "Utilisateur" -OrgName "Entreprise" -AdminPassword "P@ssw0rd"
```

## Processus de maintenance

### Cycle de mise à jour mensuel

Le maintien à jour de l'image Master suit un processus mensuel rigoureux :

1. **Préparation**
   - Démarrage de la VM Master sur Proxmox
   - Vérification et sauvegarde de l'image actuelle

2. **Mise à jour des logiciels**

```powershell
choco upgrade all -y
```

3. **Mises à jour Windows**
   - Application des correctifs de sécurité
   - Installation des mises à jour cumulatives
   - Vérification de la stabilité post-mise à jour

4. **Nouvelle capture d'image Master**
   - Exécution du processus de Sysprep et capture décrit précédemment
   - Importation de la nouvelle image dans MDT
   - Mise à jour des séquences de tâches pour utiliser la nouvelle image

## Déploiement des images avec intégration AD

### Configuration de l'automatisation MDT pour l'intégration AD

1. **Modification de la séquence de tâches**
   - Ajout de l'étape d'intégration au domaine
   - Configuration des informations d'identification du domaine

```xml
<sequence>
    <!-- Autres étapes de la séquence -->
    <step type="BDD_JoinDomain" name="Join Domain">
        <defaultVarList>
            <variable name="JoinDomain" value="domaineentreprise.local"/>
            <variable name="DomainAdmin" value="AdminMDT"/>
            <variable name="DomainAdminDomain" value="DOMAINE"/>
            <variable name="DomainAdminPassword" value="MotDePasseSécurisé"/>
            <variable name="MachineObjectOU" value="OU=Ordinateurs,OU=Entreprise,DC=domaineentreprise,DC=local"/>
        </defaultVarList>
    </step>
    <!-- Suite de la séquence -->
</sequence>
```

2. **Configuration du fichier Bootstrap.ini**
   - Paramétrage des informations de connexion au partage de déploiement
   - Configuration des informations de base pour le démarrage du déploiement

```ini
[Settings]
Priority=Default

[Default]
DeployRoot=\\serveur\DeploymentShare$
UserID=MDTService
UserDomain=DOMAINE
UserPassword=MotDePasseSécurisé
SkipBDDWelcome=YES
```

### Processus de déploiement

1. **Préparation du poste client**
   - Configuration du BIOS/UEFI pour le démarrage PXE
   - Connexion au réseau via adaptateur Ethernet

2. **Démarrage en PXE et sélection du déploiement**
   - Sélection de la séquence de tâches appropriée
   - Le système déploie automatiquement l'image et configure le poste

3. **Intégration automatique à l'Active Directory**
   - Jonction au domaine avec placement dans l'OU spécifiée
   - Application des stratégies de groupe (GPO)

```powershell
# Cette étape est automatisée par MDT via la séquence de tâches
# Le code suivant est exécuté automatiquement par la séquence:
Add-Computer -DomainName "domaineentreprise.local" -Credential $DomainCredential -OUPath "OU=Ordinateurs,OU=Entreprise,DC=domaineentreprise,DC=local" -Restart
```

4. **Validation post-déploiement**
   - Vérification de l'intégration au domaine
   - Test des applications déployées
   - Validation de la connectivité réseau

## Intégration Active Directory

### Configuration du contrôleur de domaine

1. **Création d'une unité d'organisation dédiée**

```powershell
New-ADOrganizationalUnit -Name "Ordinateurs" -Path "OU=Entreprise,DC=domaineentreprise,DC=local"
```

2. **Création d'un compte de service pour MDT**

```powershell
New-ADUser -Name "MDTService" -SamAccountName "MDTService" -UserPrincipalName "MDTService@domaineentreprise.local" -Path "OU=Services,DC=domaineentreprise,DC=local" -AccountPassword (ConvertTo-SecureString "MotDePasseComplexe" -AsPlainText -Force) -Enabled $true
```

3. **Attribution des droits nécessaires**

```powershell
Add-ADGroupMember -Identity "Domain Computers" -Members "MDTService"
# Donnez à ce compte les droits de jonction au domaine:
dsacls "OU=Ordinateurs,OU=Entreprise,DC=domaineentreprise,DC=local" /G "DOMAINE\MDTService:CC;Computer"
```

### Stratégies de groupe (GPO)

1. **Création des GPO spécifiques**

```powershell
New-GPO -Name "Config-MDT-Deployed-PCs" -Comment "Configuration pour postes déployés par MDT"
New-GPLink -Name "Config-MDT-Deployed-PCs" -Target "OU=Ordinateurs,OU=Entreprise,DC=domaineentreprise,DC=local"
```

2. **Configuration des politiques**
   - Paramètres de sécurité
   - Configuration des mises à jour Windows
   - Restrictions d'accès et de logiciels

## Cas pratiques de déploiement

### Déploiement de masse

Pour un déploiement simultané sur plusieurs machines :
1. Préparation d'un switch dédié au déploiement
2. Configuration des postes avec démarrage PXE
3. Lancement du déploiement avec sélection automatique
4. Supervision via la console MDT

### Déploiement avec personnalisation

Pour adapter l'image à des cas spécifiques :
1. Utilisation de profils de déploiement différenciés
2. Paramétrage des variables de séquence de tâches
3. Application de scripts post-installation spécifiques
4. Personnalisation via les réponses automatisées (fichiers XML)

### Débogage et résolution de problèmes

En cas d'échec de déploiement :
1. Consultation des journaux MDT (C:\DeploymentShare\Logs)
2. Analyse des erreurs spécifiques
3. Test en mode manuel ou verbeux
4. Correction des séquences de tâches problématiques

## Considérations de sécurité

### Sécurisation du processus

1. **Protection des images**
   - Stockage sécurisé des images Master
   - Chiffrement des données sensibles
   - Contrôle d'accès strict aux ressources MDT

2. **Authentification**
   - Utilisation d'identifiants spécifiques pour le déploiement
   - Séparation des privilèges administratifs
   - Rotation régulière des mots de passe

3. **Audit et traçabilité**
   - Journalisation de tous les déploiements
   - Suivi des modifications d'image
   - Rapports d'activité réguliers

### Mise en conformité

- Application des politiques de sécurité de l'entreprise
- Respect des exigences réglementaires (RGPD, etc.)
- Validation périodique par l'équipe sécurité

Cette méthodologie permet d'assurer un déploiement fiable et homogène sur l'ensemble du parc informatique, tout en optimisant les ressources et le temps nécessaires à la mise en service des postes de travail. La solution proposée répond aux objectifs fixés en termes de standardisation, rapidité de déploiement, intégration à l'Active Directory et facilité de maintenance.
