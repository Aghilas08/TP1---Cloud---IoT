----
Auteur : Aghilas OULD BRAHAM
----
Groupe : RSA 2025-2026
----
<h3 align="center">Rapport : 5G Telco Cloud</h3>

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
</p>

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

###### utilisateur "aghi" avec l'id 1000 est membre du groupe sudo ce qui implique que cette utilisateur peut executer des commandes en tant que super-utilisateur.

### La configuration réseau
<p align="center">
  <img src="/img/ip.png" width="720">
  <br>
  <em>Figure 11 : informations réseau</em>
</p>

###### ens33 : (@MAC = 00:0c:29:f6:ef:f2) lui est attribuer l'addresse ip 192.168.232.132 (auprés d'un serveur dhcp)
###### lo : c'est l'interface loopback (locale) avec une ip 127.0.0.1
### Service SSH
<p align="center">
  <img src="/img/ssh2.png" width="720">
  <br>
  <em>Figure 12 : informations service ssh</em>
</p>

###### active(running) signifie que le service ssh est bien installer et en mode listen (en ecoute) sur le port par défault 22.
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
* Premieres commande a éxecuter :
````shell
sudo apt update && sudo apt upgrade -y
````

---
# Partie 2
## Free5GC
**free5GC** est un projet **open source** (par **Linux Foundation**) qui implémente le **cœur de réseau 5G** (5G Core Network) conformément aux spécifications **3GPP**.Il est conçu principalement pour la recherche et l’expérimentation sur les réseaux 5G.
  * **La Linux Foundation** est une organisation à but non lucratif qui soutient le développement de projets open source, dont Linux et de nombreuses technologies comme Kubernetes.
  * **Le 3GPP** est un partenariat international d’organismes de normalisation,leurs roles est de standardiser les protocoles et les architectures réseaux télécomes (3G, 4G, 5G,...)

### Objectif : 
 * Installation des outils essentiels :
   * docker, kind, kubectl
 * Préparer l'environnement pour Free5GC :
   * installer le noyau GTP5G.
 * Création du cluster Kubernetes.

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


  * Étape 3 : **L'exécution** (Le conteneur s'exécute de manière isolée sur le système hôte et partage le **kernel** du système d'exploitation hôte (contrairement aux VM))
### Installation
*	Configurer le **Docker apt repository** :

<p align="center">
  <img src="/img/Image1.png" width="720">
  <br>
  <em>Figure 17 : Docker repository</em>
</p>

* Instalation du package Docker :
````bash
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
````
* Verification et teste :
  
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
  <em>Figure 21 : processus GTP5G</em>
</p>

* **gtp5g** : Module chargé, taille 151552 octets, avec 0 utilisateurs actifs pour le moment
* **udp_tunnel** : Module dépendance chargé, taille 32768 octets, utilisé par 1 module (**gtp5g**)
* pas des **PID** de processus. Les modules du noyau ne sont pas des processus utilisateur, ils font partie intégrante du noyau Linux.

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
  <em>Figure 22 : kind</em>
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

### Ajout des CNI plugins
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
  <em>Figure 24 : CNI plugins</em>
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
  <em>Figure 25 : Architecture du cluster</em>
</p>

* **Composants du plan de contrôle :**
  * **kube-apiserver :** C’est le point d’entrée central pour toutes les commandes kubectl, les communications internes et externes (Il s'agit du front-end pour le plan de contrôle Kubernetes).
  * **etcd :** Base de données clé-valeur consistante et hautement disponible utilisée comme mémoire de sauvegarde pour toutes les données du cluste.
  * **kube-scheduler :** Il a pour rôle de surveiller les pods nouvellement créés et choisir sur quel nœud chaque un pod va s’exécuter.
  * **controller-manager :**

<p align="center">
  <img src="/img/etat-cluster.png" width="780">
  <br>
  <em>Figure 26 : Etat du cluster</em>
</p>

---
---
# Glossaire
* **VM** : Virtual Machine
* **5G** : 5 Generation
* **5GC** : 5G Core Network
* **3GPP** : 3rd Generation Partnership Project
* **gNB** : Next Generation Node B
* **gtp5g** : GPRS Tunneling Protocol for 5G

---
# Ressources 
* **Figure 1** : https://techtoday.lenovo.com/fr/fr/solutions/smb/hyperviseur
* **Ducumentation free5gc** : https://free5gc.org/
* **Figure 2 --> Figure 15** : Captures d'écran
* **Figure 16** : https://www.docker.com/resources/what-container/
* **Figure 17** : https://github.com/Aghilas08/Docker.git
* **Figure 18 --> Figure 24** : Captures d'écran
* **Figure 25** : https://kubernetes.io/fr/docs/concepts/architecture/#plugins-r%C3%A9seau
