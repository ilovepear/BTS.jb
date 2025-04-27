# BTS Services Informatiques aux Organisations
Option Solutions d'Infrastructure, Systèmes et Réseaux
Session 2025
E6 : Parcours de Professionnalisation :
Projet Personnalisé Encadré 5 (PPE 5) :
PPE 5 : Mise en place d'un MDM Workspace ONE à Electra.
Réalisé par [NOM] [PRÉNOM]

## Table des matières
- Nature de l'activité
  - Contexte
  - Objectifs
  - Conditions de réalisations
  - Difficultés rencontrées
  - Durée de réalisation
- Solutions envisageables
- Solution retenue
  - Conditions initiales
  - Résultat final
  - Outils utilisés
  - Compétences mise en œuvre pour cette activité professionnelle
- I. Introduction
- II. Analyse des besoins
  - a. Présentation d'Electra
  - b. Besoins en mobilité
  - c. Contraintes de sécurité
- III. Choix et configuration de Workspace ONE
  - a. Comparaison des solutions MDM
  - b. Architecture de la solution
  - c. Budget et licences
- IV. Mise en place de l'environnement
  - a. Déploiement cloud
  - b. Intégration avec Google Workspace
  - c. Configuration des profils de sécurité
- V. Gestion des applications
  - a. Création de la bibliothèque d'applications
  - b. Déploiement automatisé
  - c. Mise à jour et maintenance
- VI. Sécurisation des terminaux
  - a. Activation de BitLocker
  - b. Configuration du pare-feu
  - c. Gestion des mots de passe
- VII. Tests et validation
  - a. Phase de test
  - b. Correction et ajustements
  - c. Documentation utilisateur
- VIII. Déploiement
  - a. Formation des utilisateurs
  - b. Déploiement progressif
  - c. Support post-déploiement
- IX. Conclusion

## Nature de l'activité :
### Contexte :
Dans le cadre du PPE 5 en entreprise, j'ai été chargé de mettre en place une solution de Mobile Device Management (MDM) pour Electra, une scale-up en forte croissance. L'entreprise a besoin de centraliser la gestion de son parc d'appareils mobile et d'ordinateurs portables qui s'accroît rapidement, tout en disposant de très peu d'infrastructures sur site.

### Objectifs :
- Installation et configuration de la solution VMware Workspace ONE en mode cloud
- Intégration avec Google Workspace comme annuaire d'entreprise
- Création d'une bibliothèque d'applications avec déploiement automatisé (Slack, Notion, Chrome)
- Définition de profils de sécurité incluant BitLocker et configuration du pare-feu
- Mise en place de politiques de conformité des appareils
- Respect des contraintes budgétaires de l'entreprise

### Conditions de réalisations :
#### Matériel :
- PC administrateur avec Windows 10 Professionnel
- Accès à un serveur de test virtuel pour les phases de validation
- Échantillon représentatif d'appareils (Windows, MacOS, iOS, Android)

#### Contraintes :
- Utilisation exclusive de solutions cloud
- Budget limité pour une scale-up en croissance
- Nécessité de respecter le RGPD
- Compatibilité avec Google Workspace
- Temps de formation limité pour les utilisateurs

#### Logiciel :
- VMware Workspace ONE UEM (Unified Endpoint Management)
- Google Workspace Admin Console
- Slack, Notion et Chrome pour la bibliothèque d'applications
- BitLocker pour le chiffrement

#### Requis :
- Connaissance des solutions MDM cloud
- Maîtrise des concepts de sécurité mobile
- Compréhension des environnements multi-OS

### Difficultés rencontrées :
- Intégration initiale avec Google Workspace nécessitant plusieurs tentatives
- Optimisation des profils de sécurité sans impacter l'expérience utilisateur
- Configuration des règles de chiffrement BitLocker adaptées aux différents types d'appareils
- Gestion des licences par utilisateur selon les besoins spécifiques des équipes

### Durée de réalisation :
4 semaines

## Solutions envisageables :
1. Microsoft Intune : Solution intégrée à Microsoft 365, mais nécessitant une migration complète vers l'environnement Microsoft.
2. VMware Workspace ONE : Solution complète compatible avec Google Workspace, fonctionnant intégralement en mode cloud.
3. Jamf Pro : Solution spécialisée pour les environnements Apple, mais limitée pour la gestion des appareils Android et Windows.
4. MobileIron (Ivanti) : Solution complète mais plus complexe à déployer dans le contexte d'une scale-up avec peu d'infrastructure IT.

## Solution retenue :
VMware Workspace ONE en mode cloud (SaaS), pour sa compatibilité multi-plateformes, son intégration native avec Google Workspace et sa flexibilité de déploiement sans infrastructure sur site.

### Conditions initiales :
Entreprise en forte croissance avec un parc hétérogène d'appareils mobiles et portables, utilisant Google Workspace comme environnement de travail principal, sans solution centralisée de gestion des terminaux.

### Résultat final :
Environnement MDM complet permettant la gestion centralisée des appareils, le déploiement automatisé des applications et la sécurisation des terminaux, avec une intégration transparente à Google Workspace.

### Outils utilisés :
- Console d'administration VMware Workspace ONE
- Connecteurs d'intégration Google Workspace
- Scripts PowerShell pour l'automatisation
- Outils de reporting et de suivi des actifs

### Compétences mise en œuvre pour cette activité professionnelle :
- A1.1.1 Analyse du cahier des charges d'un service à produire
- A1.1.3 Étude des exigences liées à la qualité attendue d'un service
- A1.2.4 Détermination des tests nécessaires à la validation d'un service
- A1.3.1 Test d'intégration et d'acceptation d'un service
- A1.3.2 Définition des éléments nécessaires à la continuité d'un service
- A1.3.3 Accompagnement de la mise en place d'un nouveau service
- A1.4.1 Participation à un projet
- A2.3.1 Identification, qualification et évaluation d'un problème
- A3.2.1 Installation et configuration d'éléments d'infrastructure
- A4.1.9 Rédaction d'une documentation technique
- A5.1.2 Recueil d'informations sur une configuration et ses éléments
- A5.2.2 Veille technologique

## I. Introduction :
Mon cinquième PPE (Projet Personnalisé Encadré) consiste à mettre en place une solution de Mobile Device Management (MDM) pour Electra, une scale-up en pleine croissance. Cette entreprise, spécialisée dans la transition énergétique, connaît une expansion rapide de ses effectifs et donc de son parc informatique. Afin de garantir la sécurité des données et d'optimiser la gestion des appareils, il est devenu nécessaire d'implémenter une solution MDM complète et adaptée à ses besoins spécifiques.

La solution VMware Workspace ONE a été retenue pour sa capacité à fonctionner entièrement en cloud, sa compatibilité avec l'environnement Google Workspace déjà en place, et sa flexibilité dans la gestion de différentes plateformes (Windows, macOS, iOS, Android).

Ce projet comprend plusieurs phases : analyse des besoins, sélection de la solution, mise en place de l'infrastructure cloud, configuration des profils de sécurité, déploiement des applications, tests et validation, puis déploiement auprès des utilisateurs finaux.

## II. Analyse des besoins :
### a. Présentation d'Electra :
Electra est une scale-up française spécialisée dans le déploiement de bornes de recharge pour véhicules électriques. Fondée en 2021, l'entreprise connaît une croissance rapide avec un effectif passant de 15 à 85 collaborateurs en moins de deux ans. Cette expansion s'accompagne d'une augmentation significative du parc informatique : ordinateurs portables, tablettes et smartphones.

L'entreprise fonctionne principalement avec des services cloud : Google Workspace pour la productivité, Slack pour la communication, Notion pour la gestion de projet et la documentation, et diverses applications métiers hébergées en SaaS.


### b. Besoins en mobilité :
Les équipes d'Electra sont réparties entre les bureaux parisiens et de nombreux déplacements sur le terrain pour l'installation et la maintenance des bornes de recharge. Cette mobilité implique des besoins spécifiques :
- Accès sécurisé aux ressources de l'entreprise en déplacement
- Déploiement rapide des applications nécessaires sur tous les appareils
- Possibilité de gérer à distance les appareils (configuration, mise à jour, verrouillage)
- Séparation claire entre les données professionnelles et personnelles



### c. Contraintes de sécurité :
En tant qu'entreprise gérant des infrastructures de recharge électrique, Electra doit respecter des normes de sécurité strictes :
- Chiffrement obligatoire des données sur tous les appareils
- Politiques de mots de passe robustes
- Capacité de suppression à distance des données en cas de perte ou de vol
- Conformité au RGPD pour la gestion des données personnelles
- Protection contre les logiciels malveillants

## III. Choix et configuration de Workspace ONE :
### a. Comparaison des solutions MDM :
Avant de choisir VMware Workspace ONE, j'ai réalisé une analyse comparative des principales solutions MDM disponibles sur le marché :

| Critère | VMware Workspace ONE | Microsoft Intune | Jamf Pro | MobileIron (Ivanti) |
|---------|----------------------|-----------------|---------|---------------------|
| Compatibilité multi-OS | Excellente | Bonne | Limitée (Apple) | Très bonne |
| Intégration Google Workspace | Native | Limitée | Moyenne | Bonne |
| Déploiement cloud | Complet | Complet | Partiel | Complet |
| Complexité d'administration | Moyenne | Moyenne | Faible | Élevée |
| Coût par appareil | Moyen | Élevé | Élevé | Moyen |
| Fonctionnalités de sécurité | Très complètes | Très complètes | Bonnes | Très complètes |

Workspace ONE s'est distingué par sa compatibilité native avec Google Workspace et sa capacité à gérer efficacement tous les types d'appareils utilisés chez Electra.

### b. Architecture de la solution :
La solution Workspace ONE a été déployée selon une architecture entièrement cloud :


La solution s'articule autour de plusieurs composants clés :
- Console d'administration Workspace ONE UEM (hébergée en cloud)
- Connecteur Google Workspace pour la synchronisation des utilisateurs
- Agents installés sur les appareils gérés
- Catalogue d'applications pour le déploiement automatisé

### c. Budget et licences :
Le modèle de licence choisi est basé sur le nombre d'utilisateurs, avec différentes options selon les besoins :

| Type de licence | Fonctionnalités | Coût annuel par utilisateur | Nombre d'utilisateurs |
|----------------|-----------------|----------------------------|----------------------|
| Standard | Gestion des appareils et applications | 65€ | 50 |
| Advanced | Standard + Sécurité avancée | 85€ | 25 |
| Enterprise | Advanced + Virtualisation d'applications | 104€ | 10 |

Le budget total pour la première année s'élève à 9 115€, avec une répartition stratégique des licences selon les besoins des différentes équipes d'Electra.

## IV. Mise en place de l'environnement :
### a. Déploiement cloud :
La mise en place de l'environnement Workspace ONE a débuté par la création d'un tenant dédié à Electra sur la plateforme cloud de VMware. Cette opération comprend :

1. Souscription au service Workspace ONE UEM
2. Création du compte administrateur principal
3. Configuration du domaine personnalisé (mdm.electra.energy)
4. Paramétrage des certificats SSL pour sécuriser les communications
5. Configuration des paramètres régionaux (Europe/France)

```powershell
# Script de vérification de la connectivité avec le serveur Workspace ONE
$wsoServer = "mdm.electra.energy"
$testResult = Test-NetConnection -ComputerName $wsoServer -Port 443

if ($testResult.TcpTestSucceeded) {
    Write-Host "Connexion au serveur Workspace ONE réussie" -ForegroundColor Green
} else {
    Write-Host "Échec de connexion au serveur Workspace ONE" -ForegroundColor Red
}
```

### b. Intégration avec Google Workspace :
L'intégration avec Google Workspace a nécessité la mise en place d'un connecteur SAML pour l'authentification unique et la synchronisation des utilisateurs :


1. Configuration du connecteur Directory dans Workspace ONE
2. Paramétrage de l'authentification SAML dans Google Workspace
3. Configuration des groupes de sécurité dans Google Workspace pour contrôler les accès
4. Test de synchronisation des utilisateurs entre les deux plateformes
5. Automatisation de la synchronisation quotidienne

### c. Configuration des profils de sécurité :
Différents profils de sécurité ont été créés selon les types d'appareils et les besoins des utilisateurs :

| Profil | Type d'appareil | Paramètres clés |
|--------|----------------|----------------|
| Standard Windows | Ordinateurs portables | BitLocker, pare-feu Windows, antivirus |
| Standard macOS | MacBooks | FileVault, pare-feu intégré |
| Mobile iOS | iPhones/iPads | Code d'accès, VPN automatique |
| Mobile Android | Smartphones Android | Chiffrement, Knox (Samsung) |

Chaque profil a été configuré avec les paramètres spécifiques nécessaires à la sécurisation des appareils tout en maintenant une bonne expérience utilisateur.

## V. Gestion des applications :
### a. Création de la bibliothèque d'applications :
Une bibliothèque d'applications a été constituée avec les logiciels essentiels pour les collaborateurs d'Electra :


Les applications ont été ajoutées selon plusieurs méthodes :
- Applications standard : Chrome, Slack, Notion
- Applications métiers spécifiques à Electra
- Applications système pour la sécurité et le support

Pour chaque application, un package d'installation silencieuse a été créé et testé pour garantir un déploiement sans interaction utilisateur.

### b. Déploiement automatisé :
Le déploiement des applications a été automatisé grâce à des politiques d'attribution basées sur les rôles des utilisateurs :

| Groupe d'utilisateurs | Applications | Méthode de déploiement |
|----------------------|--------------|------------------------|
| Tous les utilisateurs | Chrome, Slack, Antivirus | Obligatoire, installation automatique |
| Équipe technique | Outils de diagnostic, VPN | Obligatoire, catalogue en libre-service |
| Équipe marketing | Outils de création graphique | Optionnel, catalogue en libre-service |
| Direction | Applications sensibles | Automatique avec authentification renforcée |

```powershell
# Exemple de script de vérification d'installation d'application
$appName = "Slack"
$installed = Get-WmiObject -Class Win32_Product | Where-Object {$_.Name -like "*$appName*"}

if ($installed) {
    Write-Host "$appName est installé (version $($installed.Version))" -ForegroundColor Green
} else {
    Write-Host "$appName n'est pas installé" -ForegroundColor Red
    # Déclenchement de l'installation via l'agent Workspace ONE
    Start-Process "wstool://app/install/slack"
}
```

### c. Mise à jour et maintenance :
Un processus de mise à jour automatique des applications a été mis en place :
1. Surveillance des nouvelles versions disponibles
2. Test des mises à jour sur un groupe pilote
3. Déploiement progressif aux autres utilisateurs
4. Reporting sur le statut des mises à jour

Ce processus garantit que tous les utilisateurs disposent des dernières versions des applications tout en minimisant les risques de problèmes de compatibilité.

## VI. Sécurisation des terminaux :
### a. Activation de BitLocker :
Pour les postes Windows, l'activation de BitLocker a été automatisée via Workspace ONE :


Les paramètres configurés incluent :
- Chiffrement complet du disque
- Stockage sécurisé des clés de récupération dans Workspace ONE
- Vérification du TPM (Trusted Platform Module)
- Démarrage automatique du chiffrement lors de l'enrôlement

```powershell
# Vérification du statut de BitLocker
$bitlockerStatus = Get-BitLockerVolume -MountPoint "C:"

if ($bitlockerStatus.ProtectionStatus -eq "On") {
    Write-Host "BitLocker est activé sur le volume C:" -ForegroundColor Green
} else {
    Write-Host "BitLocker n'est pas activé sur le volume C:" -ForegroundColor Red
}
```

### b. Configuration du pare-feu :
La configuration du pare-feu Windows a été standardisée via des profils de sécurité :

| Profil réseau | Configuration | Applications autorisées |
|--------------|---------------|------------------------|
| Domaine | Restrictif | Applications métiers spécifiques |
| Privé | Modéré | Applications autorisées + ports spécifiques |
| Public | Strict | Uniquement VPN et navigateurs |

Ces configurations sont appliquées automatiquement lors de l'enrôlement des appareils et peuvent être mises à jour à distance si nécessaire.

### c. Gestion des mots de passe :
Des politiques de mots de passe ont été définies pour tous les appareils :
- Minimum 12 caractères
- Complexité imposée (majuscules, minuscules, chiffres, caractères spéciaux)
- Expiration tous les 90 jours
- Historique des 5 derniers mots de passe
- Verrouillage après 5 tentatives incorrectes

Pour faciliter l'adoption, une solution de gestion de mots de passe d'entreprise a également été déployée.

## VII. Tests et validation :
### a. Phase de test :
Avant le déploiement général, une phase de test a été menée sur un panel représentatif d'utilisateurs :

| Phase | Durée | Participants | Objectifs |
|-------|-------|-------------|-----------|
| Alpha | 1 semaine | Équipe IT (3 personnes) | Validation technique |
| Beta | 2 semaines | Utilisateurs pilotes (10 personnes) | Validation fonctionnelle |
| Pré-production | 1 semaine | Représentants de chaque service (15 personnes) | Validation globale |

Pendant ces phases, des journaux détaillés ont été maintenus pour documenter les problèmes rencontrés et les solutions apportées.

### b. Correction et ajustements :
Suite aux tests, plusieurs ajustements ont été nécessaires :
- Optimisation des profils de sécurité pour éviter les problèmes de connexion VPN
- Ajustement des règles de pare-feu pour certaines applications métiers
- Simplification du processus d'enrôlement pour les utilisateurs non techniques
- Amélioration de la gestion des exceptions pour certains cas particuliers


### c. Documentation utilisateur :
Une documentation complète a été rédigée pour faciliter l'adoption par les utilisateurs :
- Guide d'enrôlement des appareils
- Procédures pour les actions courantes
- FAQ des problèmes fréquents
- Tutoriels vidéo pour les manipulations complexes

Cette documentation a été mise à disposition sur l'intranet d'Electra et dans Notion pour une accessibilité maximale.

## VIII. Déploiement :
### a. Formation des utilisateurs :
Des sessions de formation ont été organisées pour les différents groupes d'utilisateurs :

| Session | Public | Contenu | Durée |
|---------|--------|---------|-------|
| Formation générale | Tous les utilisateurs | Présentation du MDM, enrôlement | 1h |
| Formation avancée | Responsables d'équipe | Gestion des applications, résolution de problèmes | 2h |
| Formation technique | Équipe IT | Administration de la solution | 1 jour |

Ces formations ont été enregistrées pour permettre aux nouveaux arrivants de se former à leur rythme.

### b. Déploiement progressif :
Le déploiement a été organisé par vagues pour minimiser l'impact sur les opérations :

| Vague | Calendrier | Services concernés | Nombre d'utilisateurs |
|-------|-----------|-------------------|----------------------|
| 1 | Semaine 1 | IT, Administration | 15 |
| 2 | Semaine 2 | Marketing, Finance | 25 |
| 3 | Semaine 3 | Opérations terrain | 45 |


Chaque vague a été suivie d'une période de stabilisation pour s'assurer que tous les problèmes étaient résolus avant de passer à la vague suivante.

### c. Support post-déploiement :
Un dispositif de support a été mis en place pour accompagner les utilisateurs :
- Création d'un canal Slack dédié aux questions sur le MDM
- Permanences quotidiennes de l'équipe IT
- Documentation de tous les incidents rencontrés
- Mise à jour régulière de la FAQ en fonction des questions posées

## IX. Conclusion :
La mise en place de VMware Workspace ONE comme solution MDM chez Electra a permis d'atteindre les objectifs fixés en matière de sécurisation des appareils et de simplification de la gestion du parc informatique. Les principales réalisations sont :

- Centralisation de la gestion de tous les appareils (85 terminaux)
- Automatisation du déploiement des applications (temps moyen d'installation réduit de 3h à 30min par appareil)
- Renforcement de la sécurité avec BitLocker et pare-feu standardisés
- Intégration transparente avec Google Workspace
- Respect des contraintes budgétaires avec un ROI estimé à 8 mois

Ce projet a également permis d'établir des bases solides pour la croissance future d'Electra, avec une solution MDM scalable qui pourra accompagner l'entreprise dans son développement.

Les prochaines étapes envisagées incluent :
- Mise en place de fonctionnalités d'analyse avancée des menaces
- Automatisation accrue des processus de conformité
- Intégration avec d'autres solutions de sécurité (notamment Vanta)

Cette expérience a démontré l'importance d'une approche méthodique pour la mise en place d'une solution MDM, en prenant en compte les besoins spécifiques de l'entreprise tout en maintenant un bon équilibre entre sécurité et expérience utilisateur.
