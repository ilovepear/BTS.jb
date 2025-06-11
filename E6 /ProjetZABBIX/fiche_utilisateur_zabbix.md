# 1. Introduction

## 1.1 Qu'est-ce que Zabbix ?

Zabbix est une solution de supervision réseau open source qui permet de surveiller l'état et les performances des serveurs, équipements réseau et applications. Dans le cadre de ce projet, Zabbix 7.0.13 a été déployé sur l'infrastructure pour superviser les serveurs de l'environnement.

Le système fonctionne selon un modèle client-serveur où :
- Le serveur Zabbix (SRV-ZAB-GUA) collecte et traite les données
- Les agents Zabbix installés sur les machines à superviser (comme SRV-DC-GUA) transmettent les informations système

## 1.2 Pourquoi la supervision est importante ?

La supervision permet de :
- Détecter proactivement les problèmes avant qu'ils n'impactent les utilisateurs
- Surveiller l'utilisation des ressources (CPU, mémoire, disque, réseau)
- Maintenir la disponibilité des services critiques
- Créer un historique des performances pour l'analyse de tendances
- Réduire les temps d'arrêt grâce à une détection rapide des incidents

## 1.3 Objectif de la fiche (utiliser Zabbix au quotidien)

Cette fiche vous guide dans l'utilisation quotidienne de Zabbix pour :
- Accéder au tableau de bord de supervision
- Vérifier l'état de santé des serveurs supervisés
- Interpréter les alertes et prendre les mesures appropriées
- Assurer un suivi efficace de l'infrastructure informatique

# 2. Accès à l'interface Zabbix

## 2.1 URL d'accès au tableau de bord

L'interface web de Zabbix est accessible via l'adresse suivante :
**http://192.168.114.50/zabbix**

Cette URL correspond au serveur Zabbix (SRV-ZAB-GUA) déployé sur le réseau local 192.168.114.0/24. Assurez-vous d'être connecté au même réseau pour accéder à l'interface.

## 2.2 Identifiants de connexion (utilisateur + mot de passe)

Les identifiants par défaut de Zabbix sont :
- **Utilisateur** : Admin
- **Mot de passe** : zabbix

Il est fortement recommandé de modifier ces identifiants par défaut après la première connexion pour des raisons de sécurité. Vous pouvez également créer des comptes utilisateurs avec des privilèges spécifiques selon les besoins.

## 2.3 Présentation rapide de l'interface (tableaux de bord, menus)

L'interface Zabbix s'organise autour de plusieurs sections principales :

**Menu principal (barre latérale gauche) :**
- **Monitoring** : Tableaux de bord, problèmes, hôtes, dernières données
- **Services** : Gestion des services métier
- **Inventory** : Inventaire des équipements
- **Reports** : Rapports et statistiques
- **Configuration** : Paramétrage des hôtes, templates, actions
- **Administration** : Gestion des utilisateurs, paramètres généraux

**Tableau de bord principal :**
- Vue d'ensemble de l'état du système
- Graphiques de performance en temps réel
- Résumé des problèmes actifs
- Widgets personnalisables selon vos besoins

# 3. Consulter l'état des serveurs

## 3.1 Où voir si un serveur est en ligne ou non

Pour consulter l'état des serveurs supervisés :

1. **Via le menu "Monitoring" > "Hosts"** : Affiche la liste complète des hôtes supervisés avec leur statut en temps réel
2. **Via le tableau de bord principal** : Les widgets affichent un résumé de l'état des hôtes
3. **Via "Monitoring" > "Problems"** : Liste les problèmes actifs sur l'infrastructure

Dans la vue "Hosts", vous verrez pour chaque serveur :
- Le nom de l'hôte (ex: SRV-DC-GUA)
- L'adresse IP (ex: 192.168.114.88)
- Le statut de disponibilité de l'agent Zabbix
- Les templates appliqués (ex: Template OS Windows by Zabbix agent)

## 3.2 Signification des statuts : 🟢 Disponible, 🔴 Problème, ⚪ Inconnu

Les différents statuts d'un hôte dans Zabbix :

**🟢 Disponible (ZBX vert) :**
- L'agent Zabbix répond correctement
- La communication entre le serveur et l'agent fonctionne
- Les données sont collectées normalement
- Le serveur est considéré comme opérationnel

**🔴 Problème (ZBX rouge) :**
- L'agent Zabbix ne répond pas
- Problème de connectivité réseau
- Service Zabbix Agent arrêté sur l'hôte distant
- Pare-feu bloquant la communication (port 10050)

**⚪ Inconnu (ZBX gris) :**
- Statut indéterminé, souvent lors de la première configuration
- Problème de configuration (hostname incorrect, IP erronée)
- Agent récemment installé mais pas encore synchronisé
- Problème temporaire de résolution DNS ou réseau

## 3.3 Que faire si un hôte est en erreur ?

Procédure de diagnostic lorsqu'un hôte apparaît en erreur :

**Étape 1 - Vérification réseau :**
- Tester la connectivité avec `ping [IP_de_l'hôte]`
- Vérifier que le port 10050 est accessible avec `telnet [IP_de_l'hôte] 10050`

**Étape 2 - Vérification de l'agent :**
- Sur l'hôte distant, vérifier que le service Zabbix Agent est démarré
- Contrôler les logs de l'agent pour identifier d'éventuelles erreurs
- Vérifier la configuration de l'agent (fichier zabbix_agentd.conf)

**Étape 3 - Vérification de la configuration Zabbix :**
- Confirmer que l'IP de l'hôte est correcte dans la configuration
- Vérifier que le hostname correspond entre l'agent et la configuration serveur
- S'assurer que les templates appropriés sont appliqués

**Étape 4 - Test manuel :**
- Utiliser `zabbix_get` depuis le serveur pour tester la collecte de données
- Exemple : `zabbix_get -s [IP_hôte] -k agent.ping`

# 4. Analyser les alertes

## 4.1 Accéder à la liste des alertes en temps réel

Pour consulter les alertes actives dans Zabbix :

**Menu "Monitoring" > "Problems" :**
- Affiche tous les problèmes non résolus
- Tri possible par gravité, hôte, ou date
- Filtrage par période ou type de problème
- Vue en temps réel avec rafraîchissement automatique

**Tableau de bord principal :**
- Widget "Problem hosts" : résumé des hôtes avec problèmes
- Widget "Problems" : liste des derniers problèmes détectés
- Notifications visuelles et sonores configurables

**Menu "Monitoring" > "Events" :**
- Historique complet des événements (problèmes et résolutions)
- Permet l'analyse des tendances et récurrences

## 4.2 Distinguer les alertes critiques, majeures, mineures

Zabbix classe les alertes selon plusieurs niveaux de gravité :

**Critique (Disaster) :**
- Panne majeure affectant la disponibilité du service
- Exemples : serveur inaccessible, service critique arrêté, disque plein
- Nécessite une intervention immédiate
- Peut déclencher des notifications urgentes (SMS, appel)

**Haute (High) :**
- Problème important mais service encore partiellement fonctionnel
- Exemples : utilisation disque > 90%, charge CPU excessive prolongée
- Intervention requise rapidement pour éviter l'escalade

**Moyenne (Average) :**
- Problème modéré nécessitant une surveillance accrue
- Exemples : utilisation mémoire élevée, temps de réponse dégradé
- Planifier une intervention dans les heures suivantes

**Attention (Warning) :**
- Alerte préventive ou seuil de vigilance atteint
- Exemples : utilisation disque > 80%, charge réseau inhabituelle
- Surveillance renforcée recommandée

**Information :**
- Messages informatifs sans impact sur le fonctionnement
- Exemples : redémarrage planifié, maintenance programmée
- Aucune action immédiate requise

## 4.3 Savoir quand une alerte se résout automatiquement

Les alertes dans Zabbix se résolvent automatiquement lorsque :

**Conditions de résolution :**
- La valeur surveillée repasse sous le seuil défini
- Le service redevient accessible après une panne
- Les conditions qui ont déclenché l'alerte ne sont plus présentes
- Le délai de rétablissement configuré est respecté

**Indicateurs de résolution :**
- L'alerte disparaît de la liste "Problems"
- Un événement "OK" apparaît dans l'historique
- Le statut de l'hôte repasse au vert
- Notification de résolution envoyée (si configurée)

**Suivi des résolutions :**
- Menu "Events" : permet de voir l'historique complet problème/résolution
- Durée des incidents calculée automatiquement
- Possibilité d'ajouter des commentaires sur les interventions réalisées

**Alertes nécessitant une action manuelle :**
- Certaines alertes peuvent nécessiter un acquittement manuel
- Les problèmes structurels (panne matérielle) ne se résolvent qu'après intervention
- Les alertes de maintenance doivent être fermées manuellement après validation

# 5. Voir les données de surveillance

## 5.1 Consulter l'utilisation CPU, RAM, espace disque

**Accès aux métriques système :**
- Connectez-vous à l'interface Web Zabbix via votre navigateur
- Naviguez vers **Monitoring > Hosts** pour voir la liste des machines supervisées
- Cliquez sur le nom d'un hôte pour accéder à ses détails
- Sélectionnez l'onglet **Latest data** pour voir les dernières valeurs collectées

**Métriques clés à surveiller :**
- **CPU** : Pourcentage d'utilisation processeur, charge système (load average)
- **RAM** : Mémoire utilisée/disponible, pourcentage d'occupation mémoire
- **Espace disque** : Utilisation des partitions en Go et pourcentage, espace libre restant

**Visualisation graphique :**
- Cliquez sur **Graphs** dans le menu latéral pour voir les graphiques historiques
- Les graphiques montrent l'évolution des métriques sur différentes périodes (1h, 24h, 7j, 1mois)
- Utilisez les boutons de zoom pour ajuster la période d'affichage

## 5.2 Suivre l'état des services (ex : Active Directory, DNS)

**Surveillance des services critiques :**
- Accédez à **Monitoring > Problems** pour voir les alertes actives
- Les services supervisés incluent généralement :
  - Active Directory (port 389/636, authentification LDAP)
  - DNS (résolution de noms, temps de réponse)
  - Services Windows/Linux spécifiques
  - Connectivité réseau et ping

**Indicateurs d'état :**
- 🔴 **Rouge** : Service indisponible ou en erreur critique
- 🟡 **Jaune** : Avertissement, performance dégradée
- 🟢 **Vert** : Service fonctionnel, valeurs normales
- **Gris** : Statut inconnu ou données non disponibles

**Vérification détaillée :**
- Dans **Latest data**, filtrez par type d'item pour voir spécifiquement les services
- Consultez les triggers associés pour comprendre les seuils d'alerte configurés

## 5.3 Filtrer par machine ou par date

**Filtrage par machine :**
- Utilisez la barre de recherche dans **Hosts** pour trouver une machine spécifique
- Les filtres **Host groups** permettent de regrouper par type (serveurs, postes de travail, réseau)
- Sélectionnez plusieurs hôtes avec Ctrl+clic pour une vue comparative

**Filtrage temporel :**
- Dans les graphiques, utilisez les boutons de période prédéfinis (Last hour, Last 24 hours, etc.)
- Pour une période personnalisée, cliquez sur **Custom** et sélectionnez les dates de début/fin
- L'option **From** et **To** permet de définir une plage horaire précise

**Filtres avancés :**
- **Item type** : Filtrer par type de métrique (CPU, mémoire, réseau, etc.)
- **Application** : Regrouper les éléments par catégorie applicative
- **Tag filters** : Utiliser les étiquettes pour des recherches spécifiques

# 6. FAQ / Problèmes courants

## 6.1 Pourquoi un hôte reste "gris" (inconnu) ?

**Causes principales :**
- **Agent Zabbix non démarré** : Le service Zabbix Agent n'est pas en cours d'exécution sur la machine cible
- **Problème réseau** : Connectivité interrompue entre le serveur Zabbix et l'hôte supervisé
- **Configuration firewall** : Le port 10050 (agent Zabbix) est bloqué par un pare-feu
- **Configuration erronée** : Adresse IP incorrecte ou nom d'hôte non résolvable dans la configuration

**Diagnostic étape par étape :**
1. Vérifier la connectivité réseau : `ping [adresse_hôte]`
2. Tester le port Zabbix : `telnet [adresse_hôte] 10050`
3. Contrôler l'état du service Zabbix Agent sur l'hôte cible
4. Vérifier les logs du serveur Zabbix pour identifier les erreurs de communication

**Résolution :**
- Redémarrer le service Zabbix Agent sur l'hôte
- Corriger la configuration réseau ou firewall
- Vérifier et corriger l'adresse IP/nom d'hôte dans la configuration Zabbix
- Contrôler les autorisations et la configuration de l'agent

## 6.2 Réparations en cas d'inaccessibilité à l'interface Zabbix Web

**Vérifications préliminaires :**
- Tester l'accès depuis un autre poste de travail pour isoler le problème
- Vérifier la connectivité réseau vers le serveur Zabbix
- Contrôler si d'autres services web fonctionnent normalement

**Problèmes côté serveur :**
- **Service web arrêté** : Redémarrer Apache/Nginx sur le serveur Zabbix
- **Base de données inaccessible** : Vérifier l'état du service MySQL/PostgreSQL
- **Espace disque saturé** : Contrôler l'espace disponible sur les partitions système
- **Surcharge serveur** : Vérifier l'utilisation CPU et mémoire du serveur

**Problèmes côté client :**
- Vider le cache du navigateur et supprimer les cookies Zabbix
- Essayer un autre navigateur ou mode navigation privée
- Vérifier les paramètres proxy si applicable
- Contrôler les restrictions de sécurité réseau locales

**Actions de récupération :**
- Accès via SSH au serveur pour diagnostic système
- Consultation des logs web (/var/log/apache2/ ou /var/log/nginx/)
- Vérification des logs Zabbix (/var/log/zabbix/)
- Redémarrage des services si nécessaire

## 6.3 Que signifie "Zabbix Agent is unavailable" ?

**Signification de l'erreur :**
Cette erreur indique que le serveur Zabbix ne peut pas communiquer avec l'agent Zabbix installé sur l'hôte supervisé. La collecte de données est interrompue pour cet hôte.

**Causes techniques fréquentes :**
- **Service agent arrêté** : Le processus zabbix-agent ne fonctionne plus sur l'hôte cible
- **Problème de connectivité** : Coupure réseau temporaire ou permanente
- **Saturation réseau** : Latence élevée causant des timeouts de connexion
- **Reconfiguration système** : Changement d'adresse IP ou de configuration réseau

**Diagnostic approfondi :**
- Vérifier le statut du service : `systemctl status zabbix-agent` (Linux) ou Services Windows
- Consulter les logs de l'agent : `/var/log/zabbix/zabbix_agentd.log`
- Tester la communication : `zabbix_get -s [IP_hôte] -k system.uptime`
- Contrôler la configuration de l'agent : `/etc/zabbix/zabbix_agentd.conf`

**Résolution progressive :**
1. Redémarrer le service Zabbix Agent
2. Vérifier et corriger la configuration si nécessaire
3. Contrôler les paramètres firewall et réseau
4. Réinstaller l'agent en cas de corruption
5. Mettre à jour la configuration côté serveur si l'hôte a changé d'IP

## 6.4 Que faire si je ne vois pas un hôte que je devrais superviser ?

**Vérifications dans l'interface :**
- Contrôler les filtres actifs dans la vue **Hosts** (groupes, statuts)
- Vérifier si l'hôte est dans un groupe d'hôtes non affiché
- Rechercher par nom complet ou partiel dans la barre de recherche
- Contrôler les permissions utilisateur pour l'accès à certains groupes d'hôtes

**Problèmes de configuration :**
- **Hôte non créé** : L'hôte n'a pas été ajouté à la configuration Zabbix
- **Hôte désactivé** : L'hôte existe mais est marqué comme inactif
- **Groupe non assigné** : L'hôte n'est associé à aucun groupe d'hôtes visible
- **Templates manquants** : Aucun template de surveillance n'est lié à l'hôte

**Processus de résolution :**
1. **Vérifier l'existence** : Chercher dans **Configuration > Hosts** avec tous les filtres désactivés
2. **Contrôler l'état** : Vérifier si l'hôte est enabled/disabled
3. **Valider la configuration** : Contrôler l'adresse IP, les groupes, les templates associés
4. **Tester la connectivité** : Utiliser le bouton "Test" lors de la configuration
5. **Créer si nécessaire** : Ajouter l'hôte avec la configuration appropriée

**Création d'un nouvel hôte :**
- **Configuration > Hosts > Create host**
- Renseigner le nom d'hôte, l'adresse IP, le groupe d'appartenance
- Associer les templates de surveillance appropriés
- Configurer les macros si nécessaire
- Activer la surveillance après validation de la configuration
