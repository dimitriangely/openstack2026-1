# OpenStack 2026.1 — Référence des services et projets

> **Version de référence :** OpenStack 2026.1  
> **Objectif :** disposer d'une vue structurée des principaux projets OpenStack, de leurs responsabilités et de leur intérêt pour un laboratoire local Vagrant / VirtualBox / Ansible.

## 1. Périmètre et précautions

OpenStack est une plateforme composée de projets indépendants qui collaborent via des API et des services d'infrastructure. Tous les projets listés dans cette fiche ne sont **pas obligatoires** dans chaque déploiement.

Cette fiche distingue :
- les composants centraux d'un cloud IaaS ;
- les services de stockage et de données ;
- les services réseau ;
- l'orchestration et le provisionnement ;
- la télémétrie, l'optimisation et les fonctions spécialisées ;
- les dépendances système qui ne sont pas elles-mêmes des projets OpenStack.

La liste ci-dessous est une **sélection organisée des projets connus et couramment rencontrés**, et non une affirmation que le nombre de projets ou de services OpenStack est fixe. Pour la liste exhaustive et le statut de chaque projet, consulter le catalogue officiel OpenStack 2026.1.

## 2. Composants fondamentaux

| Service / projet | Nom technique | Responsabilité principale | Priorité pédagogique |
|---|---|---|---|
| Identity | Keystone (`keystone`) | Authentification, utilisateurs, projets, rôles, jetons et catalogue des services. | Essentielle |
| Image | Glance (`glance`) | Catalogue et mise à disposition des images de machines virtuelles. | Essentielle |
| Compute | Nova (`nova`) | Cycle de vie des instances : création, planification, gestion et suppression. | Essentielle |
| Networking | Neutron (`neutron`) | Réseaux, sous-réseaux, ports, routeurs, adresses IP et connectivité des instances. | Essentielle |
| Placement | Placement (`placement`) | Inventaire et allocation des ressources, notamment CPU et mémoire, pour le placement des instances. | Essentielle |
| Dashboard | Horizon (`horizon`) | Interface web d'administration et d'utilisation d'OpenStack. | Recommandée |
| Block Storage | Cinder (`cinder`) | Volumes de stockage bloc persistants pouvant être attachés aux instances. | Recommandée |
| Object Storage | Swift (`swift`) | Stockage distribué d'objets, accessible par API. | Selon le besoin |

### Parcours type : créer une instance

1. **Keystone** authentifie l'utilisateur et lui fournit un jeton d'accès.
2. **Nova** reçoit la demande de création de l'instance.
3. **Placement**, utilisé par le planificateur Nova, expose les ressources disponibles et leurs allocations.
4. **Glance** fournit l'image de démarrage choisie.
5. **Neutron** fournit les ports et la connectivité réseau.
6. **Cinder** peut fournir un volume persistant si le scénario le demande.
7. **Horizon** peut servir d'interface utilisateur pour déclencher et suivre ces opérations.

Le rôle exact de chaque composant dépend du scénario et de la configuration du cloud.

## 3. Stockage et données

| Service / projet | Nom technique | Responsabilité principale | Quand l'utiliser |
|---|---|---|---|
| Block Storage | Cinder (`cinder`) | Volumes persistants en mode bloc. | Disques additionnels pour les instances, stockage persistant. |
| Object Storage | Swift (`swift`) | Stockage distribué d'objets et de fichiers via API. | Données non structurées, stockage objet. |
| Shared File Systems | Manila (`manila`) | Provisionnement de partages de fichiers accessibles par plusieurs clients. | Besoin de NFS/partages gérés en tant que service, selon les pilotes disponibles. |
| Database as a Service | Trove (`trove`) | Provisionnement et gestion de bases de données en tant que service. | Fournir des bases de données aux utilisateurs du cloud. |
| Key Manager | Barbican (`barbican`) | Gestion d'éléments secrets, de clés et de certificats via API. | Besoins de gestion centralisée de secrets et d'intégration avec d'autres services. |

### Ne pas confondre Cinder, Swift et Manila

- **Cinder** : un volume bloc, comparable à un disque que l'on attache à une VM.
- **Swift** : un service de stockage objet, accessible par API.
- **Manila** : un service de partage de fichiers, conçu pour qu'un ou plusieurs clients accèdent à un système de fichiers partagé.

Les fonctionnalités réellement disponibles dépendent des pilotes de stockage et de la configuration.

## 4. Réseau et connectivité

| Service / projet | Nom technique | Responsabilité principale |
|---|---|---|
| Networking | Neutron (`neutron`) | Réseaux virtuels, sous-réseaux, ports, routeurs et connectivité. |
| DNS | Designate (`designate`) | Gestion des zones et enregistrements DNS via API. |
| Load Balancing | Octavia (`octavia`) | Mise à disposition de répartiteurs de charge. |
| Messaging | Zaqar (`zaqar`) | Service de messagerie et de files de messages accessible aux applications. |

Neutron peut être étendu par des composants ou projets additionnels. Leur disponibilité, leur maintenance et leur compatibilité doivent être vérifiées dans la documentation de la version choisie. Exemples rencontrés selon les déploiements : routage dynamique, VPN-as-a-Service et fonctions de pare-feu réseau. Ne pas supposer que ces extensions sont installées par défaut.

## 5. Orchestration et provisionnement

| Service / projet | Nom technique | Responsabilité principale |
|---|---|---|
| Orchestration | Heat (`heat`) | Orchestration de ressources OpenStack à partir de modèles. |
| Workflow | Mistral (`mistral`) | Définition et exécution de workflows automatisés. |
| Container Infrastructure Management | Magnum (`magnum`) | Provisionnement d'infrastructures de conteneurs, selon les pilotes pris en charge. |
| Bare Metal | Ironic (`ironic`) | Provisionnement et gestion de serveurs physiques. |

### Heat, Ansible et Terraform : rôles différents

- **Heat** orchestre principalement des ressources dans OpenStack à partir de modèles.
- **Ansible** automatise la configuration des systèmes et des applications, par exemple l'installation et la configuration des services sur `controller`, `compute1` et `block1`.
- **Terraform** gère le cycle de vie de ressources déclarées via des fournisseurs, dont OpenStack.

Ces outils peuvent être complémentaires ; ils ne sont pas des substituts parfaitement équivalents.

## 6. Télémétrie, optimisation et coûts

| Service / projet | Nom technique | Responsabilité principale |
|---|---|---|
| Telemetry Data Collection | Ceilometer (`ceilometer`) | Collecte de mesures et d'événements liés aux ressources cloud. |
| Telemetry Alarming | Aodh (`aodh`) | Déclenchement d'alarmes selon des critères configurés. |
| Infrastructure Optimization | Watcher (`watcher`) | Analyse et recommandations d'optimisation des ressources. |
| Rating | CloudKitty (`cloudkitty`) | Mesure et notation de l'utilisation des ressources, notamment pour des scénarios de refacturation. |
| Instance High Availability | Masakari (`masakari`) | Gestion de certains scénarios de haute disponibilité des instances. |

La présence et l'intérêt de ces composants dépendent des besoins opérationnels. Ils ne sont généralement pas requis pour un premier déploiement de laboratoire.

## 7. Autres projets spécialisés

| Service / projet | Nom technique | Responsabilité principale |
|---|---|---|
| Accelerator Life Cycle Management | Cyborg (`cyborg`) | Gestion d'accélérateurs matériels compatibles, selon l'intégration et les pilotes. |
| Resource Reservation | Blazar (`blazar`) | Réservation de ressources cloud pour des périodes définies. |

Les projets spécialisés peuvent avoir des exigences particulières en matière de matériel, de pilotes ou d'intégration. Vérifier leur statut et leur documentation pour OpenStack 2026.1 avant de les inclure dans un déploiement.

## 8. Dépendances système : utiles, mais distinctes des projets OpenStack

| Service système | Exemple de nom de service | Utilité |
|---|---|---|
| Base de données | `mariadb` | Stockage des données d'état de nombreux services OpenStack. |
| Broker de messages | `rabbitmq-server` | Communication asynchrone entre composants, selon l'architecture. |
| Open vSwitch | `openvswitch-switch` | Commutation virtuelle utilisée dans certaines configurations réseau. |
| Synchronisation de l'heure | `chrony` | Maintien d'une heure cohérente entre les nœuds. |
| Serveur web / proxy API | `apache2` ou autre selon l'installation | Peut héberger ou relayer des API et interfaces, selon la configuration. |

Un **service OpenStack** est une fonctionnalité ou un composant d'un projet OpenStack. Une **unité systemd** est un service géré par le système Linux. Les deux notions ne sont pas interchangeables.

Exemple :
- `neutron-openvswitch-agent` : agent du projet Neutron ;
- `openvswitch-switch` : service système fournissant l'infrastructure Open vSwitch.

## 9. Parcours recommandé pour le laboratoire Vagrant

Pour un laboratoire avec les machines `controller`, `compute1` et `block1` suivre un parcours progressif est plus formateur que de tenter d'installer tous les projets en même temps.

| Étape | Composants | Objectif d'apprentissage |
|---:|---|---|
| 1 | Keystone | Comprendre utilisateurs, projets, rôles, jetons et catalogue de services. |
| 2 | Glance | Comprendre les images utilisées pour démarrer les instances. |
| 3 | Placement | Comprendre l'inventaire et l'allocation des ressources. |
| 4 | Nova | Comprendre le planificateur, les services de calcul et le cycle de vie des instances. |
| 5 | Neutron + Open vSwitch | Comprendre les réseaux virtuels, les agents réseau et la connectivité. |
| 6 | Cinder | Comprendre les volumes, LVM et iSCSI dans un scénario adapté. |
| 7 | Horizon | Administrer le cloud à travers une interface web. |
| 8 | Services additionnels | Ajouter un service uniquement lorsqu'un besoin pédagogique ou technique le justifie. |

Ce tableau est un **ordre pédagogique**, pas une séquence stricte d'installation. Il faut respecter les dépendances et l'ordre prescrits par le guide officiel de la version utilisée.

## 10. Références officielles

- [Catalogue des projets OpenStack 2026.1](https://docs.openstack.org/2026.1/projects.html)
- [Guide d'installation OpenStack 2026.1](https://docs.openstack.org/2026.1/install/)
- [Documentation Cinder 2026.1](https://docs.openstack.org/cinder/2026.1/)
- [Documentation Nova](https://docs.openstack.org/nova/latest/)
- [Documentation Neutron](https://docs.openstack.org/neutron/latest/)

> **Règle pratique :** alignez les guides de Nova, Neutron, Keystone, Glance, Placement et Cinder sur la même version OpenStack. Ne pas mélanger des instructions de différentes branches sans vérifier les différences de configuration et de compatibilité.
