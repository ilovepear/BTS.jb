# Table des matières
Nature de l'activité
Contexte
Objectifs
Conditions de réalisations
Matériel
Contraintes
Logiciel
Requis
Difficultés rencontrées 
Durée de réalisation 
Solutions envisageables
Solution retenue 
Conditions initiales 
Résultat final
Outils utilisés 
Compétences mise en œuvre pour cette activité professionnelle 
I. Introduction 
II. Préparation du poste 
a. Sélection du matériel adéquat 
b. Préparation serveur WDS
c. Mastérisation 
d. BitLocker 
e. Installation des certificats d'entreprise
III. Vérification de la conformité 
a. Test de conformité 
IV. Prise de rendez-vous
V. Mise à jour de la base de données
VI. Installation et configuration du poste
a. Configuration utilisateur
b. Installation des logiciels spécifiques
c. Paramétrages réseau 
d. Configuration messagerie 
VII. Conclusion 

# Nature de l'activité :
## Contexte :
Dans le cadre entreprise, je dois effectuer la préparation et l'installation d'un nouveau poste informatique pour un utilisateur de l'entreprise Oodrive en suivant les procédures établies.

## Objectifs :
Installation du système d'exploitation (Windows 11 Professionnel), configuration de la sécurité (BitLocker et Crowdstrike), installation des certificats d'entreprise et vérification de conformité du poste aux standards Oodrive.

## Conditions de réalisations :
### Matériel :
Dell Latitude 7340 (pour les développeurs) :
- Processeur Intel Core i7 de 13ème génération
- 32 Go RAM DDR5
- 512 Go SSD NVMe

Dell Latitude 7320 (pour les autres collaborateurs) :
- Processeur Intel Core i5 de 13ème génération
- 16 Go RAM DDR5
- 256 Go SSD NVMe

### Contraintes :
- Utilisation d'un environnement Windows
- Respect des standards de sécurité Oodrive
- Intégration au domaine d'entreprise

### Logiciel :
- Windows 10 Professionnel
- Microsoft Office 365
- BitLocker
- Agent FusionInventory
- Certificats wifi d'entreprise

### Requis :
- Accès au serveur WDS/MDT
- Réseau filaire pour la masterisation
- Droits administrateur pour l'installation

### Difficultés rencontrées :
Aucune difficulté majeure rencontrée lors de cette intervention.

### Durée de réalisation :
1H00

## Solutions envisageables :
Installation via serveur WDS/MDT pour le déploiement du système d'exploitation et configuration manuelle des paramètres spécifiques.

## Solution retenue :
Utilisation du serveur WDS avec le protocole MDT pour la masterisation du poste et configuration automatisée.

### Conditions initiales :
Installation et configuration complète d'un poste sous Windows 10 Professionnel, paramétrage de BitLocker et installation des certificats d'entreprise et des logiciels standards Oodrive.

### Résultat final :
Poste informatique conforme aux standards Oodrive, intégré au domaine et pleinement fonctionnel pour l'utilisateur.

### Outils utilisés :
Serveur WDS (Windows Deployment Services), MDT (Microsoft Deployment Toolkit), FusionInventory, PowerShell et Windows 10 Professionnel.

## Compétences mise en œuvre pour cette activité professionnelle :
- Prise en charge d'incidents et de demandes d'assistance liés au domaine de spécialité du candidat
- Élaboration de documents relatifs à la production et à la fourniture de services
- Productions relatives à la mise en place d'un dispositif de veille technologique et à l'étude d'une technologie, d'un composant, d'un outil ou d'une méthode
- A.1.1.1 Analyse du cahier des charges d'un service à produire
- A.1.1.3 Étude des exigences liées à la qualité attendue d'un service
- A.1.2.4 Détermination des tests nécessaires à la validation d'un service
- A.1.3.1 Test d'intégration et d'acceptation d'un service
- A.1.3.2 Définition des éléments nécessaires à la continuité d'un service
- A.1.3.3 Accompagnement de la mise en place d'un nouveau service
- A.1.4.1 Participation à un projet
- A.2.1.1 Accompagnement des utilisateurs dans la prise en main d'un service
- A.2.2.1 Suivi et résolution d'incidents
- A.2.2.2 Suivi et réponse à des demandes d'assistance
- A.3.1.3 Suivi d'incidents et de remédiations
- A.3.2.2 Remplacement ou mise à jour d'éléments défectueux ou obsolètes
- A.3.3.2 Planification des sauvegardes et gestion des restaurations
- A.4.1.9 Rédaction d'une documentation technique
- A.5.1.2 Recueil d'informations sur une configuration et ses éléments

# I. Introduction :
Mon cinquième PPE (Projet Personnalisé Encadré) consiste à effectuer la préparation et l'installation d'un poste informatique pour un nouvel utilisateur chez Oodrive. Ce processus implique la sélection du matériel approprié selon le profil de l'utilisateur, la masterisation du poste via le serveur WDS avec MDT, la configuration de la sécurité et l'installation des logiciels standards de l'entreprise.

# II. Préparation du poste :
## a. Sélection du matériel adéquat :

Chez Oodrive, nous disposons de deux modèles principaux de postes informatiques attribués selon le profil métier de l'utilisateur :

- Dell Latitude 7340 pour les développeurs : ces postes sont équipés de 32 Go de RAM pour supporter les environnements de développement qui nécessitent davantage de ressources.
- Dell Latitude 7320 pour les autres collaborateurs : ces postes sont équipés de 16 Go de RAM, suffisants pour les tâches bureautiques standard.

Dans le cadre de cette intervention, l'utilisateur étant un développeur backend, j'ai sélectionné un Dell Latitude 7340 avec 32 Go de RAM.


Chaque poste est identifiable via son numéro de série et son identifiant dans l'inventaire :
- Numéro de série : 3HMZXV3
- ID Inventaire : LDP-2501 (L pour Laptop, D pour Dell, P pour Paris, 25 pour l'année et 01 pour le premier de l'année)

## b. Préparation serveur WDS :

Le serveur WDS (Windows Deployment Services) couplé à MDT (Microsoft Deployment Toolkit) est l'outil principal de déploiement des postes chez Oodrive. Il permet de centraliser les images système et d'automatiser le processus d'installation.

Pour préparer le déploiement, je me connecte à l'interface d'administration du serveur WDS pour vérifier la disponibilité des images et m'assurer que le service est opérationnel.

Dans la console MDT, je vérifie que l'image système Windows 10 Professionnel est bien à jour avec les derniers correctifs de sécurité et les logiciels standards Oodrive préinstallés.

L'image système contient déjà une série de logiciels préinstallés conformément aux standards de l'entreprise :
- Suite Microsoft Office 365
- Agent FusionInventory pour la gestion de parc
- Certificats wifi d'entreprise
- Outils de développement standards (pour les postes développeurs)

## c. Mastérisation :

Pour masteriser le poste informatique, je dois le connecter au réseau d'entreprise via un câble Ethernet pour établir une connexion avec le serveur WDS.


Je démarre le poste et j'appuie sur la touche F12 pendant le démarrage pour accéder au menu de démarrage (Boot Menu). Je sélectionne l'option "Network Boot" (PXE Boot) pour démarrer sur le réseau, et je désactive le Secure Boot.

![Menu de démarrage - Boot sur PXE](Figure 5 Menu de démarrage - Boot sur PXE)

Le poste se connecte au serveur WDS et affiche la liste des images disponibles. Je sélectionne l'image "Windows 10 Pro - Oodrive Standard Build v2.3" qui correspond au standard actuel de l'entreprise.


L'installation du système d'exploitation débute. Cette étape dure environ 45 minutes pendant lesquelles le système est installé, les pilotes sont configurés et les logiciels standards sont préinstallés.


Pendant le processus d'installation, le système effectue automatiquement les tâches suivantes :
- Installation de Windows 11 Professionnel
- Installation des pilotes spécifiques au modèle Dell Latitude 7340
- Installation des logiciels standards (Office 365, navigateurs, etc.)
- Configuration des paramètres de sécurité de base
- Intégration au domaine Oodrive
- Création d'un compte administrateur local temporaire

## d. BitLocker :

Une fois l'installation terminée, je vérifie et configure le chiffrement BitLocker pour sécuriser les données du poste. Je me connecte avec le compte administrateur local et j'ouvre une fenêtre PowerShell en mode administrateur.

Je commence par vérifier l'état actuel du chiffrement avec la commande suivante :

```
manage-bde -status C:
```

Comme le chiffrement n'est pas encore activé, je procède à son activation avec la commande :

```
manage-bde -on C: -RecoveryPassword -RecoveryKey D:\BitLockerRecovery\
```

Cette commande active le chiffrement BitLocker sur le lecteur C:, génère un mot de passe de récupération et sauvegarde la clé de récupération dans le dossier spécifié.


Conformément à la politique de sécurité Oodrive, je m'assure que la clé de récupération est également sauvegardée dans l'Active Directory de l'entreprise depuis mon poste en RDP, en accédant au serveur AD.
```

Sinon,je démarre le chiffrement BitLocker démarre et s'exécutera en arrière-plan. Il prendra plusieurs heures pour se terminer complètement, mais le poste reste utilisable pendant ce processus.

## e. Installation des certificats d'entreprise :

Pour permettre au poste de se connecter aux réseaux wifi sécurisés de l'entreprise, je dois installer les certificats d'entreprise. Ces certificats sont préinstallés dans l'image mais je vérifie leur bonne installation.

Pour ce faire, j'ouvre la console de gestion des certificats (certmgr.msc) et je vérifie la présence des certificats Oodrive dans le magasin "Autorités de certification racines de confiance".

Je vérifie également la configuration des profils wifi en ouvrant les paramètres réseau et en m'assurant que les profils "Oodrive-Dev" est correctement configuré.

# III. Vérification de la conformité :
## a. Test de conformité :

Une fois l'installation et la configuration du poste terminées, je dois effectuer des tests de conformité pour m'assurer que le poste respecte les standards Oodrive :

- Système d'exploitation : version à jour avec les derniers correctifs
- BitLocker : chiffrement activé et fonctionnel
- Agent FusionInventory : installé et communicant avec le serveur
- Certificats d'entreprise : correctement installés
- Intégration au domaine : vérification de l'appartenance au domaine Oodrive

Je vérifie donc à la fois sur le poste et dans mon AD pour plus de facilités quand j'en configure plusieurs en même temps.


Je vérifie particulièrement les points suivants qui sont essentiels pour la sécurité et le bon fonctionnement du poste :

1. Statut du chiffrement BitLocker : 100% (Terminé)
2. État des mises à jour Windows : À jour
3. Configuration du pare-feu : Activé avec les règles Oodrive
4. Agent FusionInventory : Connecté et fonctionnel
5. Appartenance au domaine : Vérifié

Tous les points de conformité étant validés, je peux poursuivre avec la préparation du poste pour l'utilisateur final.

# IV. Prise de rendez-vous :

Une fois le poste préparé et vérifié, je dois planifier la livraison et l'installation du poste avec l'utilisateur final. Pour cela, j'utilise un modèle d'email standardisé :

```
Objet : Livraison de votre nouveau poste informatique - [Date]

Bonjour [Prénom de l'utilisateur],

J'ai le plaisir de vous informer que votre nouveau poste informatique est prêt à être installé.

Je vous propose un rendez-vous le [date] de [heure début] à [heure fin] pour procéder à l'installation.

Pour faciliter la mise en place de votre nouveau poste, merci de :
- Préparer la liste des logiciels spécifiques dont vous avez besoin
- Sauvegarder vos données importantes sur votre espace OoDrive
- Répertorier vos favoris de navigateur si nécessaire
- Noter les configurations spécifiques à votre environnement de travail

Je vous accompagnerai pendant l'installation pour m'assurer que tout fonctionne correctement et répondre à vos questions.

Merci de me confirmer si ce créneau vous convient ou de me proposer une alternative.

Cordialement,
[Prénom Nom]
Service Informatique Oodrive
```

Je programme ce rendez-vous en tenant compte des disponibilités de l'utilisateur et de la charge de travail du service informatique.

# V. Mise à jour de la base de données :

Après avoir préparé le poste et avant de le livrer à l'utilisateur, je dois mettre à jour la base de données de gestion de parc pour affecter le poste à son futur utilisateur.

Chez Oodrive, nous utilisons FusionInventory couplé à GLPI pour la gestion de notre parc informatique. Je me connecte à l'interface web de GLPI pour effectuer la mise à jour.

Dans l'interface, je recherche le poste par son numéro de série (3HMZXV3) et j'accède à sa fiche. Je modifie ensuite les informations suivantes :

- État : "En production" (au lieu de "En préparation")
- Utilisateur attribué : [Nom et prénom de l'utilisateur]
- Localisation : [Bureau/étage de l'utilisateur]
- Date de mise en service : [Date du jour]
- Commentaire : "Nouveau poste - Dell Latitude 7340 - Profil développeur"


Je sauvegarde ces modifications pour mettre à jour la base de données. Cette étape est importante pour le suivi du parc informatique et pour les futures interventions sur ce poste.

# VI. Installation et configuration du poste :
## a. Configuration utilisateur :

Lors du rendez-vous avec l'utilisateur, je procède à la configuration finale du poste. Je commence par créer son profil utilisateur en me connectant avec son compte domaine.

La première connexion initialise son profil et configure automatiquement les paramètres de base. Je vérifie que l'utilisateur a bien accès à ses ressources réseau :

- Lecteurs réseaux mappés correctement
- Accès à la messagerie et à l'agenda
- Connexion aux imprimantes de proximité

Je vérifie également que les stratégies de groupe (GPO) s'appliquent correctement en exécutant la commande :

```
gpupdate /force
```

## b. Installation des logiciels spécifiques :

En fonction des besoins spécifiques de l'utilisateur, j'installe les logiciels supplémentaires nécessaires à son activité. Pour un développeur backend, j'installe généralement :

- Visual Studio Code
- Git
- Docker Desktop
- Environnements de développement spécifiques (Node.js, Python, etc.)
- Outils de base de données

L'installation de ces logiciels se fait soit manuellement, soit via FusionInventory qui permet de déployer des logiciels à distance.

Pour les logiciels sous licence, je vérifie que les licences sont bien attribuées à l'utilisateur dans notre système de gestion de licences.

## c. Paramétrages réseau :

Je configure les paramètres réseau spécifiques à l'environnement de travail de l'utilisateur :

- Configuration du VPN d'entreprise pour le travail à distance
- Vérification des accès aux serveurs de développement
- Configuration des profils wifi avec les certificats d'entreprise
- Test de connectivité avec les différentes ressources réseau

Pour un développeur, je m'assure également que les accès aux environnements de développement, de test et de production sont correctement configurés selon ses droits.

## d. Configuration messagerie :

Je configure le client de messagerie Outlook avec le compte professionnel de l'utilisateur. L'intégration au domaine facilite cette étape car les informations nécessaires sont automatiquement récupérées depuis l'Active Directory.

Je vérifie que l'utilisateur a accès à sa boîte mail, son calendrier et ses contacts. Je configure également les signatures électroniques conformément à la charte graphique d'Oodrive.

# VII. Conclusion :

Ce projet m'a permis de mettre en pratique les compétences acquises en préparation et déploiement de postes informatiques en environnement professionnel. J'ai pu suivre l'ensemble du processus, de la sélection du matériel adapté au profil utilisateur jusqu'à la livraison d'un poste entièrement configuré et conforme aux standards de l'entreprise Oodrive.

L'utilisation des outils de déploiement automatisé (WDS/MDT) a considérablement réduit le temps de préparation tout en garantissant une configuration homogène et conforme aux exigences de sécurité. La mise à jour de la base de données de gestion de parc assure une traçabilité complète du matériel.

Cette expérience m'a également permis de développer mes compétences en relation client lors de la livraison du poste et de la formation de l'utilisateur à son nouvel environnement de travail.
