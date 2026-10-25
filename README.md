# Readme

## Objectif du lab

Ce lab consiste à déployer une infrastructure cloud OpenStack en suivant la documentation officielle, tout en automatisant les différentes étapes avec **Vagrant, VirtualBox et Ansible (`ansible_local`)**.

L’objectif est de comprendre le fonctionnement interne d’OpenStack plutôt que de déléguer l’installation complète à un outil de déploiement intégré.

Pour un parcours en infrastructure, cloud et DevSecOps, cet exercice permet d’étudier les composants d’un cloud privé, leurs dépendances, leur configuration et les mécanismes nécessaires à leur sécurisation.

Documentation officielle : [OpenStack 2026.1](https://docs.openstack.org/2026.1/).

---

## 1. Comprendre les éléments du lab

### 1.1 OpenStack 2026.1 (Gazpacho)

OpenStack est une plateforme open source permettant de construire et d’exploiter un cloud privé ou public.

Elle permet notamment de :

- Créer des machines virtuelles à la demande.
- Gérer des réseaux virtuels et des adresses IP.
- Créer et attacher des volumes de stockage.
- Gérer les utilisateurs, les projets et les autorisations.
- Exposer des API pour automatiser les opérations du cloud.

Par exemple, une entreprise peut utiliser OpenStack pour permettre à ses équipes de créer leurs propres machines virtuelles sans devoir demander manuellement à un administrateur de leur installer un serveur.

### 1.2 Installation hyrbide (manuelle + automatiser)

L’installation suit le guide officiel et que les services sont configurés progressivement, au lieu de confier tout le déploiement à une solution qui automatise l’ensemble.

Pour installer le service d’identité Keystone, les étapes peuvent comprendre :

1. Installer les paquets requis.
2. Configurer la base de données.
3. Créer la base de données et les permissions associées.
4. Configurer le service Keystone.
5. Initialiser sa base de données.
6. Démarrer et activer le service.
7. Vérifier son fonctionnement.

Une installation "hybride" ne signifie pas nécessairement qu’il faut taper chaque commande soi-même : Ansible permet automatiser les étapes, tout en conservant une organisation qui permet de comprendre chacune d’elles.

**Intérêt pédagogique :** comprendre ce que chaque commande configure, pourquoi elle est nécessaire et quelles dépendances existent entre les services.

---

## 2. À quoi servent Vagrant, VirtualBox et Ansible ?

Ces trois outils ont des rôles différents et complémentaires.

### 2.1 Vagrant : créer et gérer l’environnement de laboratoire

Vagrant automatise la création et la configuration initiale des machines virtuelles.

Les caractéristiques des machines peuvent être décrites dans un fichier `Vagrantfile`. On peut notamment y définir :

- Une machine virtuelle `controller`, `compute1`, `bloc1`.
- Le système d’exploitation de chaque machine.
- La mémoire vive et le nombre de processeurs virtuels.
- Les interfaces réseau.
- Les opérations de provisionnement à exécuter.

Commandes utiles :

```powershell
vagrant up
```

Démarre les machines décrites dans le `Vagrantfile` et crée celles qui n’existent pas encore, si nécessaire.

```powershell
vagrant status
```

Affiche l’état des machines gérées par Vagrant.

**Vagrant ne déploie pas OpenStack à lui seul.** Il prépare et gère les machines (ou noeuds/nodes) sur lesquelles OpenStack sera installé.

### 2.2 VirtualBox : fournir la virtualisation

VirtualBox est l’hyperviseur du laboratoire. Il permet de faire fonctionner plusieurs systèmes Linux isolés sur un ordinateur Windows.

Architecture simplifiée :

```text
Ordinateur physique
└── Windows 11
    └── VirtualBox
        ├── VM controller
        │   └── Services de contrôle OpenStack
        │
        |── VM compute1
        |    └── Services de calcul OpenStack
        │
        └── VM block1
            └── Services de stockage bloc OpenStack

```

Chaque machine virtuelle possède son propre système d’exploitation, ses processus, ses interfaces réseau et ses ressources allouées.

Il faut distinguer deux niveaux de virtualisation :

- **VirtualBox** exécute les machines virtuelles Linux du laboratoire.
- **OpenStack Nova** gère les instances que l’on crée ensuite dans le cloud.

Dans un laboratoire imbriqué, les instances OpenStack peuvent être exécutées avec QEMU en mode émulation logicielle si la virtualisation matérielle imbriquée n’est pas disponible. Cette solution est plus lente, mais peut suffire pour apprendre les mécanismes fondamentaux d’OpenStack.

### 2.3 Ansible : automatiser l’installation des services

Ansible automatise les opérations qui seraient autrement effectuées manuellement.

Il utilise des fichiers YAML appelés *playbooks* pour décrire les tâches à exécuter sur les machines.

Exemple de playbook :

```yaml
- name: Installer RabbitMQ
  ansible.builtin.apt:
    name: rabbitmq-server
    state: present

- name: S'assurer que RabbitMQ est démarré et activé
  ansible.builtin.service:
    name: rabbitmq-server
    state: started
    enabled: true
```

Cet exemple installe les paquets indiqués sur une machine Debian ou Ubuntu. Il ne constitue pas, à lui seul, une installation complète d’OpenStack : d’autres tâches sont nécessaires pour configurer les services, leurs bases de données, leurs fichiers de configuration et leur démarrage.

---

## 3. Que signifie `ansible_local` ?

Ansible peut être exécuté de plusieurs façons.

### 3.1 Mode classique : depuis une machine de contrôle

Dans une architecture classique, Ansible est installé sur une machine de contrôle. Celle-ci se connecte aux serveurs Linux via SSH et leur envoie les tâches à exécuter.

```text
Machine de contrôle
      │
      │ SSH
      ▼
┌──────────────┐
│ Serveur Linux│
└──────────────┘
```

### 3.2 Mode `ansible_local` : depuis la machine virtuelle

Avec le provisionneur `ansible_local` de Vagrant, Ansible est exécuté **à l’intérieur de la machine virtuelle cible**. Vagrant peut installer Ansible dans cette machine, puis lancer les playbooks depuis celle-ci.

```text
Ordinateur Windows
│
├── Vagrant
│
└── VirtualBox
    │
    ├── VM controller
    │   ├── Ansible
    │   └── Services OpenStack
    │
    └── VM compute1
        ├── Ansible
        └── Services OpenStack
```

Cette représentation est indicative : la présence d’Ansible sur chaque VM dépend de la configuration réelle du `Vagrantfile`, dans mon cas j'ai choisi ansible_local (voir les avantages ci-dessous).

Les avantages possibles :

- Ne pas avoir à installer Ansible sur Windows.
- Exécuter les tâches depuis l’intérieur de la VM.
- Conserver les playbooks dans le dépôt du projet.
- Automatiser le provisionnement avec Vagrant.

**`ansible_local` n’est pas un composant d’OpenStack.** C’est une manière d’exécuter les automatisations.

Il ne faut pas non plus le confondre avec **OpenStack-Ansible**, un projet distinct qui fournit une méthode intégrée de déploiement d’OpenStack.

---

## 4. Comment les outils fonctionnent-ils ensemble ?

Prenons un scénario dans lequel les services de contrôle sont installés sur `controller` et les services de calcul sur `compute1`.

| Étape | Outil | Action |
|---|---|---|
| 1 | Vagrant | Décrit et crée les machines virtuelles. |
| 2 | VirtualBox | Exécute les machines virtuelles Linux. |
| 3 | Vagrant | Configure le provisionnement. |
| 4 | Ansible | Prépare les systèmes et installe les dépendances. |
| 5 | Ansible | Configure les bases de données et les services d’infrastructure nécessaires. |
| 6 | Ansible | Installe et configure les composants OpenStack selon le guide officiel. |
| 7 | Ansible | Démarre les services et applique les configurations requises. |
| 8 | Commandes OpenStack | Vérifient que les services fonctionnent correctement. |

Les étapes peuvent être exécutées séparément (et l'ont été dans mon cas) afin de contrôler le résultat avant de poursuivre. Cela facilite le diagnostic.

Par exemple, si Keystone ne démarre pas, le problème peut venir :

- D’une base de données inaccessible.
- D’un fichier de configuration incorrect.
- D’un problème de résolution DNS.
- D’un mot de passe erroné.
- D’une dépendance manquante.
- D’un service qui n’a pas démarré.

Le but n’est pas simplement d’obtenir un cloud fonctionnel : **il s’agit de comprendre comment le diagnostiquer et le reconstruire.**

---

## 5. Les principaux composants OpenStack

| Composant | Fonction | Exemple |
|---|---|---|
| Keystone | Identité et authentification | Authentifier un utilisateur du cloud. |
| Glance | Gestion des images | Fournir une image Linux pour créer une VM. |
| Placement | Inventaire et allocation des ressources | Suivre les ressources disponibles pour les instances. |
| Nova | Calcul et gestion des instances | Créer et gérer les machines virtuelles. |
| Neutron | Réseau | Créer des réseaux, sous-réseaux et routeurs virtuels. |
| Cinder | Stockage en blocs | Attacher un volume persistant à une VM. |
| Horizon | Interface Web | Administrer le cloud depuis un navigateur. |

Ces services n’ont pas tous le même statut : certains sont fondamentaux pour un déploiement donné, tandis que d’autres sont facultatifs ou dépendent des objectifs.

Horizon peut être ajouté pour disposer d’une interface graphique, mais l’administration d’OpenStack reste possible en ligne de commande. Cinder peut également être introduit ultérieurement si le stockage persistant n’est pas nécessaire aux premières validations.

### 5.1 Le contrôleur le nœud de calcul et le stockage bloc

Dans une architecture simplifiée à trois machines :

- **`controller`** héberge les services de contrôle : authentification, API, planification et autres composants centraux, selon la répartition retenue.
- **`compute1`** héberge les composants nécessaires à l’exécution des instances, notamment l’agent de calcul Nova et l’hyperviseur utilisé.
- **`block1`** stocke leurs disques persistants (volumes).

Le contrôleur participe à la décision de placement d’une instance. Le nœud de calcul fournit les ressources nécessaires à son exécution. Le noeud de stockage fourni un volume persistant, si l’instance est supprimé, le volume et ses données restent. On peux alors le rattacher à une autre instance.

---

## 6. Installation guidée ou OpenStack-Ansible ?

Il faut distinguer deux approches.

### Approche 1 : installation guidée et automatisée progressivement

Je suit le guide officiel et j'automatise chaque étape avec mes propres playbooks Ansible afin de comprendre les dépendances et les paramètres de chaque service.

### Approche 2 : OpenStack-Ansible

OpenStack-Ansible est un projet qui fournit une méthode de déploiement et d’exploitation d’OpenStack fondée sur Ansible. Il automatise une grande partie du déploiement selon son architecture, ses rôles et ses procédures.

| Critère | Installation guidée | OpenStack-Ansible |
|---|---|---|
| Objectif principal | Apprendre le fonctionnement interne des services | Déployer OpenStack selon une méthode intégrée |
| Contrôle pédagogique | Très élevé | Élevé, avec une partie de la logique déjà abstraite |
| Configuration | Importante, service par service | Une grande partie est prise en charge par les rôles existants |
| Compréhension des dépendances | Étude progressive | Compréhension de l’architecture et du fonctionnement des rôles |
| Intérêt pour un laboratoire | Excellent pour les fondamentaux | Excellent pour étudier une méthode de déploiement industrialisée |

L’installation guidée est pertinente pour comprendre les mécanismes sous-jacents. Elle ne signifie toutefois pas que l’architecture obtenue est automatiquement adaptée à la production.

---

## 7. Intérêt du lab

Ce lab permet de travailler plusieurs compétences utiles.

### 7.1 Automatisation et Infrastructure as Code

J'apprend à décrire l’infrastructure et sa configuration dans des fichiers versionnés :

- **Vagrant** décrit l’environnement des machines virtuelles.
- **Ansible** décrit les configurations et les opérations à effectuer.
- **Git** peut conserver l’historique des changements.

Cela se rapproche des pratiques d’Infrastructure as Code, même si Vagrant et Ansible n’ont pas exactement les mêmes responsabilités que Terraform.

### 7.2 Administration système Linux

Les manipulations couvrent des éléments fondamentaux :

- Paquets et dépendances.
- Services `systemd`.
- Permissions et propriétaires.
- Fichiers de configuration.
- Réseau et résolution de noms.
- Journaux et diagnostic.

Ces compétences sont utiles en administration système, en cloud et en Devops/DevSecOps.

### 7.3 Sécurité et durcissement

Une fois le déploiement fonctionnel, le laboratoire peut être enrichi avec des contrôles de sécurité.

| Domaine | Contrôle envisageable |
|---|---|
| Identités | Vérifier les comptes, rôles et permissions. |
| Secrets | Éviter de stocker les mots de passe en clair dans Git. |
| Réseau | Limiter les ports exposés et contrôler les flux entre les nœuds. |
| Système | Vérifier les permissions des fichiers de configuration. |
| Services | Vérifier l’état des services et leur configuration. |
| Journalisation | Contrôler les journaux système et ceux des services OpenStack. |
| Automatisation | Tester les playbooks, leur idempotence et leurs effets. |
| Vulnérabilités | Inventorier les paquets et vérifier les mises à jour de sécurité. |

Il est également possible d’intégrer des contrôles dans une pipeline CI/CD : validation YAML, analyse statique, détection de secrets et vérification de certaines configurations.

---

## 8. Validation du bon fonctionnement

L’installation ne doit pas être considérée comme terminée uniquement parce que les playbooks se sont exécutés sans erreur. Des validations fonctionnelles sont nécessaires.

### Niveau 1 — Infrastructure

- Les deux VM démarrent.
- Les interfaces réseau sont correctement configurées.
- La résolution de noms fonctionne.
- Les communications nécessaires entre les nœuds sont possibles.
- Les ressources CPU et RAM sont suffisantes pour le scénario choisi.

### Niveau 2 — Services OpenStack

- Les services nécessaires sont actifs.
- Les API répondent.
- Les identifiants et les points de terminaison sont configurés.
- L’authentification fonctionne.
- Les journaux ne révèlent pas d’erreur bloquante.

### Niveau 3 — Fonctionnalités du cloud

Si les composants correspondants ont été installés et configurés, les tests peuvent comprendre :

1. Authentifier un utilisateur.
2. Afficher les services OpenStack.
3. Importer une image Linux.
4. Créer un réseau virtuel.
5. Créer une instance.
6. Vérifier que l’instance démarre.
7. Tester sa connectivité réseau.

### Niveau 4 — Reproductibilité

Il est particulièrement utile de vérifier que l’on peut :

- Relancer un playbook sans provoquer de modifications inutiles.
- Identifier précisément les étapes qui échouent.
- Reconstruire une VM à partir de la configuration.
- Documenter les dépendances et les prérequis.
- Contrôler les changements apportés à l’infrastructure.

Il faut distinguer la reproductibilité d’une machine virtuelle de celle de l’ensemble du cloud : recréer une VM ne signifie pas nécessairement que les données, les identités et tous les services OpenStack seront restaurés automatiquement.

---

## 9. Dimensionnement du laboratoire

Un ordinateur doté de bonne capacité mémoire **64 Go de RAM dans mon cas** constitue une bonne base pour ce type de laboratoire. Plusieurs facteurs doivent néanmoins être pris en compte :

- La RAM réellement disponible pour les VM.
- Le nombre de processeurs virtuels alloués.
- L’espace disque disponible.
- La compatibilité de VirtualBox avec les mécanismes de virtualisation de Windows.
- La capacité à faire fonctionner des instances OpenStack à l’intérieur des VM.

OpenStack est une plateforme relativement lourde. Il est donc préférable de dimensionner le laboratoire avec soin et de commencer par une installation minimale.

Si la virtualisation matérielle imbriquée n’est pas disponible, le recours à QEMU en mode émulation logicielle peut permettre certains tests, mais les performances seront nettement inférieures à celles d’un environnement bénéficiant de la virtualisation matérielle.

Cela ne remet pas en cause l’intérêt pédagogique du lab.

---

## Conclusion

Ce lab consiste à construire un cloud OpenStack en séparant clairement trois responsabilités :

- **Vagrant** : préparer et gérer les machines virtuelles.
- **VirtualBox** : exécuter les machines virtuelles.
- **Ansible avec `ansible_local`** : automatiser, depuis les VM, les étapes d’installation et de configuration d’OpenStack.

Le guide officiel sert de référence pour comprendre chaque service, tandis que l’automatisation rend les opérations plus reproductibles.

Il est utile de conserver une trace de chaque étape : commandes exécutées, fichiers modifiés, dépendances, erreurs rencontrées, tests fonctionnels et contrôles de sécurité.

**La véritable valeur de ce lab est de pouvoir expliquer comment le cloud fonctionne, comment il est construit et comment vérifier qu’il fonctionne correctement et de manière sécurisée.**
