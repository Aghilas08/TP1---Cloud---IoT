----
Auteur : Aghilas OULD BRAHAM
Groupe : RSA 2025-2026
----
<h3 align="center">Rapport : UPC Réseau 5G</h3>

<p align="center"><i>Simulation 5G Core Network </i></p>
<p align="center">
    <a href="https://www.u-paris.fr/">
       <img alt="Université Paris Cité" src="https://upload.wikimedia.org/wikipedia/commons/thumb/4/4b/Logo_Universit%C3%A9_Paris-Cit%C3%A9_%28partenariat_Wikim%C3%A9dia%29.svg/1024px-Logo_Universit%C3%A9_Paris-Cit%C3%A9_%28partenariat_Wikim%C3%A9dia%29.svg.png" width="100">
    </a>
</p>

<p align="center">
    <a href="https://www.vmware.com/">
       <img alt="VMware version" src="https://img.shields.io/badge/VMware-v17-607078?logo=vmware&logoColor=white">
    </a>
    <a href="https://ubuntu.com/">
       <img alt="Ubuntu version" src="https://img.shields.io/badge/Ubuntu-22.04%20LTS-E95420?logo=ubuntu&logoColor=white">
    </a>
    <a href="https://www.docker.com/">
      <img alt="Docker version" src="https://img.shields.io/badge/Docker-28.0-blue?logo=docker&logoColor=white">
    </a>
    <a href="https://kind.sigs.k8s.io/">
      <img alt="Kind version" src="https://img.shields.io/badge/Kind-0.30.0-orange?logo=kubernetes&logoColor=white">
    </a>
  <a href="https://helm.sh/">
    <img alt="Helm version" src="https://img.shields.io/badge/Helm-4.0-0F1689?logo=helm&logoColor=white">
  </a>
<a href="https://kubernetes.io/docs/reference/kubectl/">
   <img alt="kubectl version" src="https://img.shields.io/badge/kubectl-1.33.5-blue?logo=kubernetes&logoColor=white">
</a>


</p>

<p align="center"><i>TP proposer par : Mr BERROUBACHE Willem</i></p>
<p align="center"><i>Réaliser par : OULD BRAHAM Aghilas </i></p>

---

# Introduction
Ce tp a pour objectif de déployer et expérimenter un cœur de **réseau 5G** (5G Core Network) dans **un environnement entièrement virtualisé**. En utilisant Free5GC, une implémentation open-source conforme aux normes 3GPP,cela nous permet de comprendre le fonctionnement de l'architecture Service-Based Architecture qui caractérise les réseaux de cinquième génération.

Le déploiement utilise des outils de **conteneurisation** et **d’orchestration** comme Docker, Kubernetes (avec Kind) et Helm. Tous les composants du cœur 5G (AMF, SMF, UPF, AUSF, UDM, PCF, NSSF) sont installés dans un cluster Kubernetes. Les interfaces réseau 5G (N1 à N6) sont configurées grâce au **plugin Multus CNI**et au **module GTP5G**, afin de permettre le bon fonctionnement du trafic utilisateur.

Pour valider le fonctionnement complet du système, on a besoin de **simuler un terminal utilisateur** (UE) et **une station de base** (gNB), permettant de réaliser des testes de connectivité jusqu'au Internet.

---
# Partie 1

### Objectif : 
 * installer une VM linux.
 * Accéder a cette VM via SSH.

### Prérequis :
 * **OS Linux** : Ubuntu Server 22.04 LTS (LTS : stands for long-term support nous offre 5 années de mise a jour et de correctifs de sécurité)
 * **Hyperviseur** : ``VirtualBox`` ``VMware`` 
## Machine Virtuel 
<p align="center">
  <img src="/img/VM_H2.png"  width="300">
  <br>
  <em>Figure 1 : Schéma Virtualisation</em>
</p>

Cette configuration décrit un environnement de virtualisation. 
Le **hardware** (CPU, RAM, stockage et la carte réseau (NIC)) qui sont les ressources physiques de l’ordinateur hôte. Sur ce matériel tourne un **OS** Windows 11, qui gère directement ces ressources. au dessus, un **hyperviseur de type 2**, en l’occurrence **VMware**, permet de créer et d’exécuter des machines virtuelles. Chaque machine virtuelle dispose de ressources virtuelles (CPU , RAM...) fournies par l’hyperviseur.
Dans cet environnement, la VM utilise un **kernel** (noyau) **Linux** et exécute un **OS Ubuntu Server**, ce qui permet de bénéficier d’un système Linux complet tout en restant hébergé sur un ordinateur Windows.

## Installation
### 1. Récapitulatif VM sous VMware :

<p align="center">
  <img src="/img/recap_vm.png"  width="300">
  <br>
  <em>Figure 2 : Récapitulatif VM 5G_2025</em>
</p>

* **NAT** : (Network Address Translation) permet à la machine virtuelle d’accéder à Internet en partageant la connexion réseau de l’hôte.
L’hyperviseur agit comme un routeur : la VM utilise une adresse IP privée et passe par l’hôte pour communiquer avec l’extérieur.
* **CD/DVD** : contient l’image ISO du système d’exploitation, utilisée pour installer l’OS sur la machine virtuelle (a éjecter a la fin de l'installation).

### 2. Configuration pré-installation

<p align="center">
  <img src="/img/type_installation.png"  width="750">
  <br>
  <em>Figure 3 : Type d'installaion</em>
</p>

<p align="center">
  <img src="/img/config_reseau.png"  width="750">
  <br>
  <em>Figure 4 : Configuration réseau</em>
</p>

Cette machine est configurée comme un client DHCP, ce qui signifie qu’elle obtient automatiquement son adresse IPv4 et ses paramètres réseau auprès d’un serveur DHCP (**nom de l'interface : ens33**).

<p align="center">
  <img src="/img/config_stkg1.png"  width="710">
  <br>
  <em>Figure 5 : Configuration du stockage</em>
</p>

<p align="center">
  <img src="/img/config_stkg2.png"  width="710">
  <br>
  <em>Figure 6 : Récapitulatif Configuration du stockage</em>
</p>

Utiliser un disque entier (70GB) en LVM (Logical Volume Manager) consiste à regrouper tout le disque dans un groupe de volumes (Volume Group).
Cela permet de créer et gérer des volumes logiques (pour le système et données ... ) et de redimensionner facilement les partitions.
###### Périphériques physiques utilisé :
   * **ubuntu-vg** le LVM volume group
   * **ubuntu-lv** formater en **ext4** et monté au répértoire **/**
   * **/dev/sda** - 70GB (disque local)

  * **partition 1** : (BIOS grub spacer : espace réservé pour le bootloader **GRUB**)
  * **partition 2** : 2Go (formater en **ext4**, monté à **/boot** contien les fichiers nécessaires au démarrage)
  * **partition 3** : 68Go (LVM volume group ubuntu-vg ou on trouve tous les fichiers du système et des utilisateurs (/home, /var, /usr, /etc,...)

* * **Système de fichiers utilisé** : **ext4** est le système de fichiers par défaut pour les distributions Linux. Il offre une performante gestion des volumes.
* * **GRUB** : c'est le premier programme qui se lance quand on allume les machines dont leurs systemes d'exploitation est linux.

<p align="center">
  <img src="/img/ssh.png" width="710">
  <br>
  <em>Figure 7 : Open SSH</em>
</p>

* **SSH (Secure SHELL** : est un protocole de communication sécurisé qui permet de :
* * se connecter à distance à un serveur/ordinateur via le réseau
* * exécuter des commandes sur une machine distante
* * Transférer des fichiers de manière sécurisée (avec la commande ``scp``)

### 3. Phase d'instalation

<p align="center">
  <img src="/img/installation.png" width="720">
  <br>
  <em>Figure 8 : phase d'installation</em>
</p>

A la fin de l'installation de l'os on doit impérativement redémarrer la **vm** afin d'ejecter le support d'installation.

## Vérification
### Systeme et utilisateur
<p align="center">
  <img src="/img/infos.png" width="720">
  <br>
  <em>Figure 9 : informations sur le systeme</em>
</p>

<p align="center">
  <img src="/img/user.png" width="720">
  <br>
  <em>Figure 10 : informations sur utilisateur</em>
</p>

* utilisateur "aghi" avec l'id 1000 est membre du groupe *sudo* ce qui implique que cette utilisateur peut executer des commandes en tant que super-utilisateur.

### La configuration réseau
<p align="center">
  <img src="/img/ip.png" width="720">
  <br>
  <em>Figure 11 : informations réseau</em>
</p>

* ens33 : (@MAC = 00:0c:29:f6:ef:f2) lui est attribuer l'addresse ip 192.168.232.132 (auprés d'un serveur dhcp)
* lo : c'est l'interface loopback (locale) avec une ip 127.0.0.1
### Service SSH
<p align="center">
  <img src="/img/ssh2.png" width="720">
  <br>
  <em>Figure 12 : informations service ssh</em>
</p>

* **active** (running) signifie que le service ssh est bien installer et en mode listen (en ecoute) sur le port par défault 22.
#### Commande utile :
````shell
systemctl status ssh  # donne des infos sur le service
systemctl enable ssh  # activer le service
systemctl restart ssh # redemarer le service en cas de modification de sa configuration dans /etc/ssh/sshd_config
````
### Partition

<p align="center">
  <img src="/img/partition.png" width="720">
  <br>
  <em>Figure 13 : informations sur le stockage</em>
</p>

* **Rappel sur les droits dans linux**
  * ``b`` = périphérique bloc (block device)
  * ``utilisateur propriétaire`` : root (droits : lecture et écriture)
  * ``groupe propriétaire`` : disk (droits : lecture et écriture)
  * ``tous les autres`` : aucun droit

# Redirection de port NAT
Dans cette configuration, l’objectif est de permettre l’accès à la machine virtuel **5G_2025** depuis **la machine hôte** à l’aide du protocole SSH. La machine virtuelle utilise une carte réseau en mode NAT, ce qui lui permet de partager la connexion Internet de l’hôte tout en restant isolée du réseau local.

Pour établir la connexion SSH, une redirection de port (port forwarding) a été mise en place dans les paramètres NAT de VMnet8.

<p align="center">
  <img src="/img/nat.png" width="720">
  <br>
  <em>Figure 14 : Paramétres NAT dans VMware</em>
</p>

**Remarque :** on peut ne pas spécifier l'addresse ip de la machine distante. 

### Teste de connection :

<p align="center">
  <img src="/img/login.png" width="720">
  <br>
  <em>Figure 15 : Connection via ssh</em>
</p>

### Mise a jour du systéme
* **Premieres commande a éxecuter :**
````shell
sudo apt update && sudo apt upgrade -y
````

---
# Partie 2
## Free5GC
**free5GC** est un projet **open source** (par **Linux Foundation**) qui implémente le **cœur de réseau 5G** (5G Core Network) conformément aux spécifications **3GPP**.Il est conçu principalement pour la recherche et l’expérimentation sur les réseaux 5G.
  * **La Linux Foundation** est une organisation à but non lucratif qui soutient le développement de projets open source, dont Linux et de nombreuses technologies comme Kubernetes.
  * **Le 3GPP** est un partenariat international d’organismes de normalisation,leurs roles est de standardiser les protocoles et les architectures réseaux télécomes (3G, 4G, 5G,...)

### Objectif 
 * Installation des outils essentiels :
   * docker, kind, kubectl
 * Préparer l'environnement pour Free5GC :
   * installer le noyau GTP5G.
 * Création du cluster Kubernetes.
 * Plugins CNI et Multus CNI

### Prérequis
 * **VM ubuntu server** (VM installer dans la partie 1)

## Docker
**Docker** est une plateforme **open source** qui permet de créer, déployer et exécuter des applications dans des **conteneurs** (en lui offrant tout ce dont elle a besoin pour fonctionner).
**Un conteneur** est comme une boîte qui contient :
 * le code ou l'application
 * Toutes ses dépendances (bibliothèques, fichiers de configuration...)

<p align="center">
  <img src="/img/infra-docker.avif" width="720">
  <br>
  <em>Figure 16 : shéma docker</em>
</p>

* **Le fonctionnement :**
   * Étape 1 : L'**Image Docker** (Contient le système d'exploitation minimal, l'application et ses dépendances crée à partir d'un fichier **Dockerfile** ou d'un ``docker commit``)




   * Étape 2 : Le **Conteneur** (C'est une instance **en cours d'exécution** d'une image)


  * Étape 3 : **L'exécution** (Le conteneur s'exécute de manière isolée sur le système hôte et partage le **kernel** du système d'exploitation hôte (contrairement aux VM)
### Installation
*	Configurer le **Docker apt repository** :

<p align="center">
  <img src="/img/Image1.png" width="720">
  <br>
  <em>Figure 17 : Docker repository</em>
</p>

* **Instalation du package Docker :**
````bash
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
````
* **Verification et teste :**
  
<p align="center">
  <img src="/img/teste-docker.png" width="720">
  <br>
  <em>Figure 18 : teste docker</em>
</p>

## kubectl
``kubectl`` est l'outil en ligne de commande pour interagir avec **Kubernetes** (la plateforme open-source d'orchestration de conteneurs).Elles permet de :

* Déployer des applications sur un **cluster Kubernetes**
* Inspecter et gérer les ressources du cluster (pods, services, déploiements...)
* Consulter les logs.
* Exécuter des commandes dans les conteneur

### Installer et configurer kubectl
````shell
sudo apt update && sudo apt install -y apt-transport-https ca-certificates curl gnupg
sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.33/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
sudo chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo chmod 644 /etc/apt/sources.list.d/kubernetes.list
sudo apt update
sudo apt install -y kubectl
````
* Source : https://kubernetes.io/fr/docs/tasks/tools/install-kubectl/

<p align="center">
  <img src="/img/teste-kubectl.png" width="480">
  <br>
  <em>Figure 19 : teste kubectl</em>
</p>

## Le noyau GTP5G
**GTP5G** implémente le protocole GPRS Tunneling Protocol pour la 5G directement dans le noyau Linux. Il gère le **plan utilisateur** (User Plane) du réseau 5G (Les données utilisateur transitent via ces tunnels GTP).

* **Le rôle de GTP5G dans free5GC** :

   * **Encapsuler/désencapsuler** les paquets de données utilisateur.
   * **Router** le trafic entre la station de base (**gNB**) et le réseau de données.
   * Gérer les **sessions PDU** (Protocol Data Unit)

### Installation
````shell
apt install -y make gcc unzip gcc
curl -LO https://github.com/free5gc/gtp5g/archive/refs/tags/v0.8.10.zip
unzip v0.8.10 && cd gtp5g-0.8.10
make clean & make
sudo make install
````
* Source : https://github.com/WillemBerr/upc-free5gc

### Vérification et résultats

<p align="center">
  <img src="/img/gtp5g.png" width="600">
  <br>
  <em>Figure 20 : tunnel GTP5G</em>
</p>

Le module **gtp5g** a été chargé avec succès dans **le noyau Linux** et configuré pour se charger automatiquement à chaque démarrage du système grâce au fichier **/etc/modules-load.d/gtp5g.conf**, ce qui signifie que vous n'aurez pas besoin de le charger manuellement après un redémarrage.

<p align="center">
  <img src="/img/process-gtp5g.png" width="720">
  <br>
  <em>Figure 21 : GTP5G est bien charger</em>
</p>

* **gtp5g** : Module chargé, taille 151552 octets, avec 0 utilisateurs actifs pour le moment
* **udp_tunnel** : Module dépendance chargé, taille 32768 octets, utilisé par 1 module (**gtp5g**)
* pas de **PID** de processus. Les modules du noyau ne sont pas des processus utilisateur, ils font partie intégrante du noyau Linux.

## kind
**kind** est un outil permettant d'exécuter des **clusters Kubernetes locaux** en utilisant des conteneurs **Docker comme "nœuds"**. kind a été principalement conçu pour tester Kubernetes lui-même, mais peut également être utilisé pour le développement local ou l'intégration continue (CI).

### Installation à partir Release Binaries
* **Sous Linux (ubuntu x64)**
````shell
# For AMD64 / x86_64
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.30.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
````
  * Source : https://kind.sigs.k8s.io/docs/user/quick-start/#installation


<p align="center">
  <img src="/img/kind.png" width="400">
  <br>
  <em>Figure 22 : version kind</em>
</p>



## kind cluster
On choisi d’utiliser un nœud pour le plan de contrôle (**control plane**) et un nœud de travail (**worker node**)

* **fichier** : *mycluster.yaml*
````yaml
kind: Cluster                          # Type de ressource : un cluster KinD
apiVersion: kind.x-k8s.io/v1alpha4     # Version de l’API utilisée par KinD
nodes:                                 # Définir les 02 nœuds du cluster
  - role: control-plane                # 1er nœud : plan de contrôle (maître)
  - role: worker                       # 2eme nœud : nœud de travail

# SOURCE TP : https://github.com/WillemBerr/upc-free5gc
## dans le fichier mycluster.yaml du tp on constate qu'il y a des erreurs :
#### tabulation
````
````shell
## résultat attendu
aghi@arcadia:~/5G$ sudo kind get nodes
kind-control-plane
kind-worker
````
* **control-plane** : gère le cluster Kubernetes (API Server, Scheduler, Controller Manager, etcd)

* **worker node** : exécute les pods.

<p align="center">
  <img src="/img/cluster.png" width="980">
  <br>
  <em>Figure 23 : kind cluster</em>
</p>

### Architecture du cluster
* **1. Namespaces :** fournissent un mécanisme pour isoler (isolation logique) les ressources au sein d'un seul cluster.
  * **namespaces initiaux :**
    * **default :** Kubernetes inclut ce namespace afin qu'on puisse commencer à utiliser nos cluster sans avoir à créer de namespace.
    * **kube-system :** le namespace pour les objets créés par le système Kubernetes
  * **Affichages des namespaces :** ``kubectl get namespace``

* **2. Les concepts derrière Kubernetes :**

<p align="center">
  <img src="/img/arch-cluster.png" width="980">
  <br>
  <em>Figure 24 : Architecture du cluster</em>
</p>

* **Composants du plan de contrôle :**
  * **kube-apiserver :** C’est le point d’entrée central pour toutes les commandes kubectl, les communications internes et externes (Il s'agit du front-end pour le plan de contrôle Kubernetes).
  * **etcd :** Base de données clé-valeur consistante et hautement disponible utilisée comme mémoire de sauvegarde pour toutes les données du cluste.
  * **kube-scheduler :** Il a pour rôle de surveiller les pods nouvellement créés et choisir sur quel nœud chaque un pod va s’exécuter.
  * **controller-manager :** regroupe et exécute l’ensemble des contrôleurs Kubernetes, il est chargés de surveiller l’état du cluster.

<p align="center">
  <img src="/img/etat-cluster.png" width="780">
  <br>
  <em>Figure 25 : Les composants du plan de controle</em>
</p>

<p align="center">
  <img src="/img/nodes.png" width="780">
  <br>
  <em>Figure 26 : Nodes</em>
</p>

La commande ``kubectl get nodes -o wide`` permet d’obtenir des informations détaillées sur chaque noeud.

<p align="center">
  <img src="/img/etat-cluster1.png" width="780">
  <br>
  <em>Figure 27 : Etat du cluster</em>
</p>

La commande ``kubectl cluster-info`` confirme que le plan de contrôle Kubernetes est accessible via l’adresse **https://127.0.0.1:33527**, ce qui correspond au point d’accès local de l’API server exposer par kind. Elle indique également que le service **CoreDNS** est actif (pour la résolution interne des noms de services au sein du cluster).

* **Composants du nœud :**
  * **kubelet** : Un agent qui s'exécute sur chaque nœud du cluster. Il s'assure que les conteneurs fonctionnent dans un pod.
  * **Runtime** : exécute les conteneur.
  * **pod** : est la plus petite unité déployable dans Kubernetes, regroupant un ou plusieurs conteneurs (**best practice :** max de conteneur dans un seul pod = **02**) partageant le même stockage, réseau et spécifications d’exécution.

### Ajout des CNI plugins
Les **CNI plugins** sont des composants du réseau dans Kubernetes. Ils assurent la connectivité entre les pods, ainsi qu’entre les pods et le monde extérieur.le CNI définit comment un pod obtient une adresse IP, comment il se connecte au réseau, et comment la communication entre pods est gérée à l’intérieur du cluster.

* **Téléchargement de la derniere version :**
````shell
wget https://github.com/containernetworking/plugins/releases/download/v1.6.0/cni-plugins-linux-amd64-v1.6.0.tgz

## décompresser et extraire le fichier
tar -xzvf cni-plugins-linux-amd64-v1.6.0.tgz
````
* **Copier les fichiers dans le dossier /opt/cni/bin de chaque conteneur Docker**

<p align="center">
  <img src="/img/cni-plugins.png" width="980">
  <br>
  <em>Figure 28 : CNI plugins</em>
</p>

### Analyse du réseau Docker utilisé par le cluster Kind
* **Les réseaux docker** :
  *	Communication entre les conteneurs ou vers l’extérieur
  *	Types : host, bridge, none, overlay…
      *	**Host** : utiliser directement le réseau de l’hôte sur la quel tourne le conteneur.
      *	**None** : ne pas autoriser la communication entre l’intérieur et l’extérieur.
      *	**Overlay** : communiquer entre différents conteneurs situer sur diffèrent machines en un réseau unique.
      * **bridge** : C’est une couche logique connecter à l’interface réseau de la machine (ou a la **vm**), permet de créer un réseau virtuel interne à notre hôte.
  *	*Un conteneur n’a pas d’@ip fixe*.

<p align="center">
  <img src="/img/net-docker.png" width="580">
  <br>
  <em>Figure 29 : @ip des conteneurs dans docker</em>
</p>

<p align="center">
  <img src="/img/net-kind.png" width="980">
  <img src="/img/infos_net_nodes.png" width="980">
  <em>Figure 30 : réseau Docker utilisé par le cluster Kind</em>
</p>

Ce réseau est de type **bridge**, ce qui signifie qu’il permet la communication entre les conteneurs hébergés localement tout en les isolant du réseau principal de la machine hôte. L’inspection du réseau montre qu’il fonctionne sur le **sous-réseau IPv4 172.18.0.0/16** avec une passerelle **172.18.0.1**, ainsi qu’un **sous-réseau IPv6** associé.
Les deux conteneurs du cluster (kind-control-plane et kind-worker) sont connecté respectivement avec les adresses ip ``172.18.0.3`` et ``172.18.0.2``.
Le paramètre **com.docker.network.bridge.enable_ip_masquerade=true** permet la traduction d’adresses (NAT) vers l’extérieur.

### Multus CNI

Ce plugin permet à Kubernetes de gérer plusieurs interfaces réseau par pod, contrairement à la configuration standard où chaque pod ne possède qu’une seule interface gérée par le CNI principal.

<p align="center">
  <img src="/img/multus_cni_pod.png" width="680">
  <br>
  <em>Figure 31 : Multus CNI</em>
</p>

<p align="center">
  <img src="/img/install-multus.png" width="880">
  <br>
  <em>Figure 32 : installation Multus CNI</em>
</p>

* **Pourquoi installe-t-on multus dans ce TP ?**
On sait que Kubernetes n'a qu'une seule interface réseau par pod par défaut, mais **Free5GC** n’est pas une simple application c'est un système 5G complet qui comporte plusieurs interfaces réseau pour les fonction **AMF, SMF, UPF, AUSF,...** (exemple d'interface : **N1, N2 , N3...**)
---

# Partie 3
### Objectif
* Helm (installation et utilisation)
* Free5GC
  * Configurer les adresses des intarfeces (N6)
  * Déployer Free5GC avec Helm
* Diagnostiquer et corriger les erreurs :
  * problème d’image MongoDB
  * erreur ''mongo: command not found''
  * absence de PersistentVolume
* Architecture SBA

## Helm
**Helm** est un outil de gestion de packages pour Kubernetes.
* **il se compose principalement de trois éléments :**
  * **chart** : définit les métadonnées de l'application (nom, version, dépendances...). 
  * **valeurs** : définissent les remplacements de variables de manière à réutiliser le chart,autrement dit c'est un fichier ou on mets nos configuration personnalisée.
  * **répertoire de modèles (templates/)** : contient les modèles et les associe aux valeurs indiquées dans le fichier *values.yaml* pour créer des manifestes.
  
L'installation d'un chart Helm entraine la création d'une instance appelée *version*. Les charts Helm sont conservés d'une version à l'autre, ce qui signifie qu'on peut utiliser une ancienne version pour revenir à la configuration souhaitée.

### Installation (avec un script bash)
  * **1.creation du script :** copier depuis https://helm.sh/fr/docs/intro/install
  * **2.droit d'éxecution**
  * **3.éxecution du script**
  * **4. vérification** : affichage de la version installer.
  
<p align="center">
  <img src="/img/install_helm.png" width="980">
  <br>
  <em>Figure 33 : installation de helm</em>
</p>

Pour configurer **Free5GC**, on dois récupérer l’**IP** et la **Gateway** du worker sur son interface *eth0* (chaque nœud (control-plane / worker) n'a qu'une seule interface ''eth0'' et son IP attribuer depuis le réseau Docker : 172.18.0.x)

<p align="center">
  <img src="/img/eth0.png" width="880">
  <br>
  <em>Figure 34 : node worker interface eth0</em>
</p>

### Téléchargement de la chart free5g
<p align="center">
  <img src="/img/chart_free_5g(1).png" width="880">
  <br>
  <em>Figure 35 : Téléchargement de la chart free5g -erreure-</em>
</p>

* **Erreure :** git ne permet pas de cloner un dossier spécifique d’un repo, seulement le repo entier.

<p align="center">
  <img src="/img/chart_free_5g(2).png" width="880">
  <br>
  <em>Figure 36 : Téléchargement de la chart free5g -solution-</em>
</p>

* Pour éviter cette erreure plus facilement il vaut mieux utiliser **helm pull** (c'est ce qui est demander dans le tp)
source : https://github.com/free5gc/free5gc-helm/tree/main/docs

````shell
## commande ajoute un repository Helm à la configuration locale
helm repo add towards5gs 'https://raw.githubusercontent.com/Orange-OpenSource/towards5gs-helm/main/repo/'
````

<p align="center">
  <img src="/img/repo-helm.png" width="880">
  <br>
  <em>Figure 37 :  ajoute du repository Helm</em>
</p>

<p align="center">
  <img src="/img/chart_free_5g(3).png" width="880">
  <br>
  <em>Figure 38 :  Téléchargement de la chart free5g -helm pull-</em>
</p>

  * * **--version :** c'est la version qu'on a récuperer dans le resultat de la commande ``helm search repo free5g``
  * *  **--untar :** extraire directement dans le dossier **free5gc**

### Installer la chart
* Déployer et grouper (isoler) les pods de free5g dans un nouveau **namespace** : ``sudo helm install free5gc-release . -n free5gc``

<p align="center">
  <img src="/img/namespace_free5gc.png" width="580">
  <br>
  <em>Figure 39 :  namespace free5gc</em>
</p>

<p align="center">
  <img src="/img/deploiment-free5g.png">
  <br>
  <em>Figure 40 :  deploiment des pods free5g</em>
</p>

* **verifier les pods** :
<p align="center">
  <img src="/img/pods_status.png" width="880">
  <br>
  <em>Figure 41 :  etats des pods</em>
</p>

* * **Init:0/1** (pour la plupart) : Les pods attendent que leurs init containers se terminent (en rélalité il attendent que mongodb demmare correctemt)
* * **ContainerCreating (UPF)** : Le pod UPF est en cours de création
* * **mongodb-0** : Statut tronqué, il faut voir son état complet
<br>

* **voir les evenements** :
<p align="center">
  <img src="/img/mongodb_erreure.png" width="880">
  <br>
  <em>Figure 42 :  mongodb_erreure</em>
</p>

* * Kubernetes n’arrive pas à télécharger l’image Docker de MongoDB depuis Docker Hub.
* * Le PVC de MongoDB attend un volume persistant (PV : est comme un disque dur statique dans Kubernetes) compatible, mais aucun PV n’existe dans le cluster.

* **Solution** :
* **1.version debian pour mongodb** : ``tag: latest`` cette version génére un probléme c'est que la commande ``mongo`` elle n'existe plus dans les versions récentes de MongoDB
    * solution : ``nano free5gc/charts/mongodb/values.yaml``
  
<p align="center">
  <img src="/img/mongodb_config.png" width="280">
  <br>
  <em>Figure 43  :  solution probléme "mongo: command not found" </em>
</p>



  * **2.PersistantVolume** :
<p align="center">
  <img src="/img/folder.png" width="480">
</p>

<p align="center">
  <img src="/img/volume.png" width="580">
  <br>
  <em>Figure 44 : Creation du PersistentVolume </em>
</p>

<p align="center">
  <img src="/img/pvc.png" width="580">
  <br>
  <em>Figure 45 : Les volumes disponibles </em>
</p>

pour garantir la persistance des données **MongoDB** dans Kubernetes, un PersistentVolume (PV) de 8GB a été créé, qui est lier a un répertoire local **/home/kubedata** sur le nœud worker Kind. MongoDB génère automatiquement un PersistentVolumeClaim (PVC) de 6Gi qui se lie au PV via la StorageClass **standard**.

## configuration N6
L'interface **N6** est l'interface de l'**UPF** (User Plane Function) qui connecte le réseau 5G au **Data Network** (DN) externe.
**N6**= passerelle vers Internet
* ``nano free5gc/values.yaml``
<p align="center">
  <img src="/img/N6.png" width="280">
  <br>
  <em>Figure 46 :  N6 network config</em>
</p>

* ``nano free5gc/charts/free5gc-upf/values.yaml`` : assignation d'une addresse ip pour N6.
<p align="center">
  <img src="/img/n6if.png" width="280">
  <br>
  <em>Figure 47 :  N6 interface config</em>
</p>

## Deploiment de Free5GC
* **Suppression de l'ancien deploiment :**
````shell
sudo helm uninstall free5gc-release . -n free5gc

# Forcer la suppression de tous les pods
sudo kubectl delete pods --all -n free5gc --grace-period=0 --force

# Ou supprimer le namespace entier (plus radical)
sudo kubectl delete namespace free5gc
````

* **Déploiment et verification des pods :**
  * `` sudo helm install free5gc-premier . -n free5gc``

<p align="center">
  <img src="/img/deploying free5g.png" width="880">
  <br>
  <em>Figure 48 :  deploiment free5g</em>
</p>

<p align="center">
  <img src="/img/pods_status(2).png" width="880">
  <img src="/img/svc.png" width="880">
  <br>
  <em>Figure 49 : etat des pods de free5gc</em>
</p>

* **Description du pod UPF** : (voir les derniers événements)
  * ``sudo kubectl describe pod free5gc-premier-free5gc-upf-upf-56b77f55d8-t64jm -n free5gc``

<p align="center">
  <img src="/img/upf.png" width="880">
  <br>
  <em>Figure 50 :  information sur le pod UPF</em>
</p>
Le journal affiche que le conteneur upf a été créé puis démarré sans erreur, indiquant un lancement réussi du service UPF.
le pod free5gc-upf a été correctement assigné sur le nœud kind-worker et que plusieurs interfaces réseau (eth0, n3, n6, n4) lui ont été attribuées via multus.


### Rappel sur Service-Based Architecture
<p align="center">
  <img src="/img/sba.png" width="880">
</p>

* **Le cœur du réseau 5G basé sur l'architecture SBA** :
  * **AUSF** (Authentication Server Function) : gère l'authentification et l'autorisation des abonnés. Vérifie l'identité des utilisateurs lors de leur connexion au réseau 5G.
  * **AMF** (Access and Mobility Management Function) : contrôle l'accès au réseau et la mobilité des terminaux. Gère l'enregistrement, la connexion et les handovers entre cellules.
  * **DN** (Data Network) : réseau de données externe (Internet) auquel l'utilisateur accède via le réseau 5G.
  * **NSSF** (Network Slice Selection Function): sélectionne le slice appropriée selon le type de service demandé par l'utilisateur.
  * **PCF** (Policy Control Function) : définit et applique les politiques de contrôle : qualité de service (QoS), tarification, gestion des flux de données.
  * **SMF** (Session Management Function) : gère les sessions de données : établissement, modification et terminaison des connexions PDU. Alloue les adresses IP.
  * **UDM** (Unified Data Management) : stocke et gère les données d'abonnement, les profils utilisateurs et les informations d'authentification.
  * **UPF** (User Plane Function) : achemine le trafic de donné utilisateur entre le terminal et le réseau externe (DN).

---

# Partie 4

### objectifs
* Mettre en place un tunnel SSH dynamique
* Configurer l’extension FoxyProxy
* Acceder a l'interface web de free5gc

L’interface Web de Free5GC tourne à l’intérieur d’une machine virtuelle et d’un cluster isolé, et n’est pas directement accessible depuis mon navigateur (sur la machine hote).

## Configuration de l'accès à l'interface Web Free5GC via SOCKS Proxy

* **L'IP externe du cluster (worker node) :** ``172.18.0.3 ``
<p align="center">
  <img src="/img/ip_externe.png" width="880">
  <br>
  <em>Figure 51 :  ip externe</em>
</p>

### Configuration du proxy SOCKS avec FoxyProxy
* **1.Installation l'extension FoxyProxy (Firefox)**
<p align="center">
  <img src="/img/fproxy.png" width="880">
  <br>
  <em>Figure 52 : FoxyProxy </em>
</p>

* **2.Créeation d'un tunnel SSH avec Dynamic Forward depuis la machine hote**
  * *sur la machine hote :*  ``ssh -D 8080 -N aghi@127.0.0.1 -p 2222`` (ne pas fermer l'invite de commande )
* **3.activation de ce proxy pour accéder aux ressources du cluster :**
<p align="center">
  <img src="/img/ajout_proxy.png" width="880">
  <em>Figure 53 :  nouveau proxy activer </em>
</p>

* **4.Accé à : http://172.18.0.2:30500 via le navigateur :**
<p align="center">
  <img src="/img/web_free5g.png" width="880">
  <em>Figure 54 : interface web free5gc </em>
</p>

--> Login : admin || password : free5gc

* **5. les logs dans foxyproxy:**
<p align="center">
  <img src="/img/log_proxy.png" width="880">
  <em>Figure 55 : journal (log) du proxy</em>
</p>

---

# Partie 5
### Objectifs
* Installer UERANSIM avec helm.
* Ajout d'un abonné dans l'interface web free5gc.
* Tester la connexion.

### installation UERANSIM

<p align="center">
  <img src="/img/abonnee(1).png" width="880">
  <em>Figure 56 : /useransim est introuvable -impossible d'istaller useransim-</em>
</p>

* cette erreur est causé par l'absence du répértpoir **useransim** dans charts free5gc (puisque j'ai pas clonner https://github.com/free5gc/free5gc-helm.git au lieu de sa j'ai utiliser **helm pull**)

<p align="center">
  <img src="/img/useransim_helm.png" width="880">
  <em>Figure 57 : helm pull /useransim</em>
</p>

* **installation avec helm :**
  
````shell
$ pwd
/home/aghi/5G/ueransim
$ ls -l
total 32
-rw-r--r-- 1 aghi aghi  271 nov.  16 11:50 Chart.yaml
-rw-r--r-- 1 aghi aghi 1950 nov.  16 11:50 open5gs-values.yaml
-rw-r--r-- 1 aghi aghi 8617 nov.  16 11:50 README.md
drwxr-xr-x 5 aghi aghi 4096 nov.  16 11:50 templates
-rw-r--r-- 1 aghi aghi 4803 nov.  16 11:50 values.yaml

$ sudo helm -n free5gc install ueransim-premier .
````
<p align="center">
  <img src="/img/useransim_install.png" width="980">
  <em>Figure 58 : installation UERANSIM avec helm</em>
</p>

* **résultat :**
<p align="center">
  <img src="/img/ue_gnb.png" width="980">
  <em>Figure 59 : worker node complet</em>
</p>

--> le **worker node** : au finale ce neud héberge tous les pods de l'infrastructure 5G.

**UERANSIM simule** : Cela permet de tester le cœur de réseau 5G (Free5gc)
  * *UE* (User Equipment) : terminal 5G (exemple : Un smartphone)
  * *gNB* (Base Station) : Une antenne 5G

### Ajout d'un abonné
<p align="center">
  <img src="/img/ajout_abonnee.png" width="980">
  <em>Figure 60 : </em>
</p>

<p align="center">
  <img src="/img/restart_pod_ue.png" width="980">
  <em>Figure 61 : </em>
</p>

<p align="center">
  <img src="/img/ue_enregistrer.png" width="980">
  <em>Figure 62 : </em>
</p>

# Teste
* **ping UE -- UPF** :
<p align="center">
  <img src="/img/ue_ping_upf.png" width="980">
  <em>Figure 63 : ping ue -- upf OK</em>
</p>

* **ping UE -- DN (google)** :
<p align="center">
  <img src="/img/ue_ping_dn.png" width="980">
  <em>Figure 63 : ping ue -- dn erreure</em>
</p>

* **identification et résolution du probléme :**

<p align="center">
  <img src="/img/erreur_ping.png" width="980">
  <em>Figure 64 : identifier le probleme -gateway-</em>
</p>

la communication entre ue et upf fonctionne correctement, l'UPF ne peut pas router le trafic vers Internet. La cause est une configuration incorrecte de la route par défaut dans l'upf.
La route par défaut est configurée avec ``172.18.0.0`` comme destination, ce qui correspond à l'adresse du réseau Docker Kind, alors qu'elle devrait pointer vers ``172.18.0.1``, qui est l'adresse de la passerelle (gateway) permettant la sortie vers Internet (cette erreur je l'ai commis quand j'ai configurer **N6**).

  * ``nano free5gc/values.yaml``
<p align="center">
  <img src="/img/soluyion_gw.png" width="200">
</p>

  * **refaire les étapes : "Deploiment de Free5GC" de la partie 3**
  * **refaire toutes les étapes de la partie 5**

````shell
## récupere le nom du pod ue et le mettre dans une variable
$ export POD_NAME=$(sudo kubectl get pods --namespace free5gc -l "component=ue" -o jsonpath="{.items[0].metadata.name}")

$ echo $POD_NAME
ueransim-premier-ue-c7fd9b989-bs7fb
````
<p align="center">
  <img src="/img/teste_connection_ue_dn.png" width="980">
  <em>Figure 65 : connexion internet</em>
</p>

----
# Glossaire
* **CNI plugins** : Container Network Interface plugins
* **IP** : Internet Protocole
* **VM** : Virtual Machine
* **5G** : 5 Generation
* **5GC** : 5G Core Network
* **3GPP** : 3rd Generation Partnership Project
* **gNB** : Next Generation Node B
* **gtp5g** : GPRS Tunneling Protocol for 5G
* **UE** : User Equipement

---
# Ressources
* **Ducumentation free5gc** : https://free5gc.org/
* **Ducumentation docker** : https://github.com/Aghilas08/Docker.git
* **Ducumentation k8s** : https://kubernetes.io/docs/concepts/
* **Ducumentation helm** : https://helm.sh/docs/
* **Figure 1** : https://techtoday.lenovo.com/fr/fr/solutions/smb/hyperviseur
* **helm** : https://www.redhat.com/fr/topics/devops/what-is-helm
* **Figure 2 --> Figure 15** : Captures d'écran
* **Figure 16** : https://www.docker.com/resources/what-container/
* **Figure 17** : https://github.com/Aghilas08/Docker.git
* **Figure 18 --> Figure 23** : Captures d'écran
* **Figure 24** : https://kubernetes.io/fr/docs/concepts/architecture/#plugins-r%C3%A9seau
* **Figure 25 --> Figure 65** : Captures d'écran
