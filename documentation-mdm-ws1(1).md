# Documentation : Mise en place d'une solution MDM WorkSpace ONE

## Sommaire
1. [Cahier des charges](#cahier-des-charges)
2. [Configuration de la solution](#configuration-de-la-solution)
3. [Mise en place opérationnelle](#mise-en-place-opérationnelle)
4. [Pour aller plus loin](#pour-aller-plus-loin)

## Cahier des charges

### Contexte et besoins

Notre entreprise, présente sur plusieurs sites internationaux, nécessitait une solution de gestion de parc informatique répondant aux besoins suivants :

- **Gestion à distance** : Permettre à l'équipe informatique d'accéder aux différentes machines des utilisateurs quel que soit leur lieu géographique, facilitant ainsi le support technique sans déplacement physique.
- **Interopérabilité** : Assurer la compatibilité avec notre solution ITSM (IT Service Management) Fleet existante pour centraliser la gestion des incidents et des demandes.
- **Compatibilité multi-plateforme** : Prendre en charge l'ensemble des systèmes d'exploitation utilisés dans notre parc (Windows, macOS, Linux, iOS, Android).
- **Conformité de sécurité** : Vérifier à distance que chaque appareil respecte nos politiques de sécurité informatique et standards de l'entreprise.
- **Déploiement centralisé** : Permettre la distribution d'applications et de mises à jour de manière uniforme sur l'ensemble du parc.

### Solution retenue

Après analyse des différentes solutions du marché, notre choix s'est porté sur VMware WorkSpace ONE (WS1), une plateforme de gestion unifiée des terminaux (UEM) pour les raisons suivantes :

- Solution complète intégrant MDM (Mobile Device Management), MAM (Mobile Application Management) et gestion des identités.
- Architecture cloud permettant une administration centralisée indépendamment de la localisation.
- Capacités avancées d'intégration avec nos systèmes existants.
- Fonctionnalités robustes de sécurité et de conformité.
- Support technique étendu et documentation complète fournie par VMware.

### Objectifs du projet

1. Déployer WorkSpace ONE comme solution MDM principale de l'entreprise
2. Intégrer la solution avec notre ITSM Fleet
3. Configurer les profils de sécurité adaptés à nos exigences
4. Mettre en place une bibliothèque d'applications essentielles
5. Former les administrateurs système à la gestion de la plateforme
6. Déployer progressivement la solution sur l'ensemble du parc informatique

## Configuration de la solution

### Architecture et composants déployés

L'architecture de notre déploiement WorkSpace ONE comprend :

- **Console d'administration WS1** : Interface centralisée pour la gestion des appareils et des applications
- **Connecteurs d'intégration** : Modules permettant la communication avec nos services d'annuaire et ITSM
- **Agents WS1** : Clients installés sur les terminaux des utilisateurs

### Configuration des rôles administrateurs

La structure administrative a été organisée selon les principes de moindre privilège :

- **Administrateurs globaux** : Accès complet à la console, gestion des politiques et des intégrations (limité à 3 personnes)
- **Administrateurs de déploiement** : Gestion des applications et des profils (équipe de 5 personnes)
- **Administrateurs support** : Accès en lecture et capacités de dépannage de base (équipe support de 10 personnes)
- **Auditeurs** : Accès en lecture seule pour les besoins de conformité (équipe sécurité)

Chaque rôle a été configuré avec des autorisations précises, documentées dans notre référentiel interne.

### Bibliothèque d'applications

Nous avons constitué une bibliothèque d'applications essentielles, déployées automatiquement sur les appareils selon le profil utilisateur :

| Application | Système(s) | Public cible | Paramètres spécifiques |
|-------------|------------|--------------|------------------------|
| Notion | Windows, macOS, iOS, Android | Tous les utilisateurs | SSO activé, espaces de travail préconfigurés |
| Slack | Windows, macOS, iOS, Android | Tous les utilisateurs | SSO activé, canaux par défaut configurés |
| Chrome Enterprise | Windows, macOS | Tous les utilisateurs | Extensions de sécurité préinstallées, paramètres de synchronisation d'entreprise |
| Bitdefender | Windows, macOS, Linux | Tous les utilisateurs | Politiques de scan automatique, reporting centralisé |
| VPN d'entreprise | Tous OS | Utilisateurs mobiles | Connexion automatique sur réseaux non fiables |
| Suite Office | Windows, macOS, iOS, Android | Tous les utilisateurs | Configuration centralisée des comptes |

Le processus de validation et d'ajout de nouvelles applications inclut des tests de sécurité et de compatibilité avant déploiement en production.

### Profils et politiques

Nous avons établi plusieurs profils pour appliquer nos politiques de sécurité et de configuration :

#### Profil de base (tous les appareils)
- Mots de passe robustes (12 caractères minimum, complexité élevée)
- Chiffrement du stockage activé
- Mises à jour de sécurité automatiques
- Configuration du pare-feu

#### Profil des appareils mobiles
- Conteneurisation des données d'entreprise
- Restrictions d'installation d'applications
- Configuration VPN automatique
- Géolocalisation activée en cas de perte

#### Profil des postes fixes
- Configuration proxy entreprise
- Restrictions d'élévation de privilèges
- Planification des sauvegardes
- Paramètres d'économie d'énergie

#### Profil des utilisateurs sensibles (Direction, Finance)
- Authentification multifacteur obligatoire
- Restrictions de transfert de données supplémentaires
- Journalisation renforcée des activités
- Contrôles d'accès réseau plus stricts

## Mise en place opérationnelle

### Planification du déploiement

Le déploiement a été réalisé en plusieurs phases :

1. **Phase pilote** (2 semaines)
   - Déploiement sur l'équipe IT (20 utilisateurs)
   - Tests et ajustements des configurations
   - Documentation des problèmes rencontrés

2. **Phase 1** (3 semaines)
   - Déploiement sur le siège social (200 utilisateurs)
   - Formation des premiers utilisateurs
   - Mise en place du support de niveau 1

3. **Phase 2** (6 semaines)
   - Déploiement sur les sites européens (500 utilisateurs)
   - Ajustements des politiques selon les retours

4. **Phase 3** (8 semaines)
   - Déploiement sur les sites internationaux (800 utilisateurs)
   - Finalisation de l'intégration avec tous les systèmes

### Procédure d'intégration des appareils

La procédure d'intégration varie selon le type d'appareil :

#### Pour les nouveaux appareils
1. Préparation de l'appareil avec image système préconfiguré
2. Enregistrement dans la console WS1
3. Attribution à l'utilisateur final
4. Déploiement automatique des applications et profils

#### Pour les appareils existants
1. Communication préalable aux utilisateurs
2. Planification des créneaux d'intervention
3. Installation de l'agent WS1 via script de déploiement
4. Vérification de la conformité et correctifs si nécessaire

### Formation des équipes

Un plan de formation complet a été établi :

- Formation approfondie pour les administrateurs (3 jours)
- Formation technique pour l'équipe support (1 jour)
- Sessions de sensibilisation pour les utilisateurs finaux (1 heure)
- Création de guides d'utilisation et de FAQ

### Procédures de support

Un processus de support dédié a été mis en place :

1. Support de niveau 1 : Équipe helpdesk formée aux problèmes courants
2. Support de niveau 2 : Administrateurs WorkSpace ONE pour les problèmes complexes
3. Support de niveau 3 : Escalade vers VMware pour les incidents critiques

Des procédures documentées ont été créées pour les incidents les plus fréquents.

## Pour aller plus loin

### Optimisations futures

Plusieurs axes d'amélioration ont été identifiés pour les phases ultérieures :

#### Intégration avancée avec d'autres systèmes
- Connecteur SIEM pour centraliser les alertes de sécurité
- Intégration avec notre solution de gestion des actifs
- Synchronisation bidirectionnelle avec notre CMDB

#### Automatisation renforcée
- Scripts de remédiation automatique pour les problèmes courants
- Workflows d'approbation pour les demandes d'applications
- Quarantaine automatique des appareils non conformes

#### Sécurité avancée
- Déploiement du module Workspace ONE Intelligence
- Analyse comportementale des utilisateurs
- Détection avancée des menaces

### Indicateurs de performance

Pour mesurer le succès du déploiement, nous suivons les KPI suivants :

| Indicateur | Objectif | Résultat actuel |
|------------|----------|-----------------|
| Taux de couverture | 95% des appareils | 87% |
| Temps de déploiement d'une application | < 4 heures | 6 heures |
| Taux de conformité sécurité | > 98% | 96% |
| Satisfaction utilisateurs | > 8/10 | 7.5/10 |
| Incidents liés aux terminaux | Réduction de 40% | Réduction de 35% |

### Retours d'expérience

Après plusieurs mois d'utilisation, nous avons recueilli les retours suivants :

**Points positifs**
- Réduction significative du temps de déploiement des applications
- Amélioration de la visibilité sur l'état du parc
- Réduction des incidents de sécurité liés aux terminaux non conformes

**Points d'amélioration**
- Nécessité d'optimiser les profils pour réduire l'impact sur la performance des appareils plus anciens
- Besoin de simplifier certains workflows pour les administrateurs
- Amélioration nécessaire de la documentation interne

### Veille technologique

Pour maintenir notre solution à jour, nous suivons activement :
- Les mises à jour de VMware WorkSpace ONE
- Les évolutions des standards de sécurité mobile
- Les nouvelles menaces ciblant les terminaux
- Les retours d'expérience d'autres entreprises utilisant la même solution

Un comité de pilotage se réunit trimestriellement pour évaluer ces évolutions et planifier les ajustements nécessaires.

---

*Document créé dans le cadre du PPE - Mars 2025*
