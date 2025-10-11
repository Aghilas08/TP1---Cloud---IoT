----
Auteur : Aghilas OULD BRAHAM
Groupe : RSA 2025-2026
----
<h3 align="center">TP1 : Machine Virtuel</h3>

<p align="center"><i>Installation ubuntu server sous VMware</i></p>
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
### Objectif : 
 * installer une VM linux.
 * Accéder a cette VM via SSH.

### Exigence :
 * **OS Linux** : Ubuntu Server 22.04 LTS (LTS : stands for long-term support nous offre 5 années de mise a jour et de correctifs de sécurité)
 * **Hyperviseur** : ``VirtualBox`` ``VMware`` 
# Machine Virtuel 
<p align="center">
  <img src="/img/VM_H2.png"  width="300">
  <br>
  <em>Figure 1 : Schéma Virtualisation</em>
</p>

Cette configuration décrit un environnement de virtualisation. 
Le **hardware** (CPU, RAM, stockage et la carte réseau (NIC)) qui sont les ressources physiques de l’ordinateur hôte. Sur ce matériel tourne un **OS** Windows 11, qui gère directement ces ressources. au dessus, un **hyperviseur de type 2**, en l’occurrence **VMware**, permet de créer et d’exécuter des machines virtuelles. Chaque machine virtuelle dispose de ressources virtuelles (CPU , RAM...) fournies par l’hyperviseur.
Dans cet environnement, la VM utilise un **kernel** (noyau) **Linux** et exécute un **OS Ubuntu Server**, ce qui permet de bénéficier d’un système Linux complet tout en restant hébergé sur un ordinateur Windows.

# Installation
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
   * **ubuntu-vg** en LVM volume group
   * **ubuntu-lv** formater en **ext4** et monté à **/**
   * **/dev/sda** - 70GB (disque local)

  * **partition 1** : (BIOS grub spacer espace réservé pour le bootloader **GRUB**)
  * **partition 2** : 2,000 Go (formater en **ext4**, monté à **/boot** contien les fichiers nécessaires au démarrage)
  * **partition 3** : 67,997 Go (LVM volume group ubuntu-vg ou on trouve tous les fichiers du système et des utilisateurs (/home, /var, /usr, /etc, etc.)

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

### 3. Phase dinstalation

<p align="center">
  <img src="/img/installation.png" width="720">
  <br>
  <em>Figure 8 : phase d'installation</em>
</p>

A la fin la machine doit etre redémarrer.

# Vérification

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

###### utilisateur "aghi" avec l'id 1000 est membre du groupe sudo ce qui implique que cette utilisateur pour executer des commandes en tant que super-utilisateur.


<p align="center">
  <img src="/img/ip.png" width="720">
  <br>
  <em>Figure 11 : informations réseau</em>
</p>

###### ens33 : (@MAC = 00:0c:29:f6:ef:f2) lui est attribuer l'addresse ip 192.168.232.132
###### lo : c'est l'interface loopback (locale) avec une ip 127.0.0.1

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

# Redirection de port NAT
Dans cette configuration, l’objectif est de permettre l’accès à la machine virtuel **5G_2025** depuis **la machine hôte** à l’aide du protocole SSH. La machine virtuelle utilise une carte réseau en mode NAT, ce qui lui permet de partager la connexion Internet de l’hôte tout en restant isolée du réseau local.

Pour établir la connexion SSH, une redirection de port (port forwarding) a été mise en place dans les paramètres NAT de VMnet8.

<p align="center">
  <img src="/img/nat.png" width="720">
  <br>
  <em>Figure 13 : Paramétres NAT dans VMware</em>
</p>

### Teste de connection :

<p align="center">
  <img src="/img/login.png" width="720">
  <br>
  <em>Figure 14 : Connection via ssh</em>
</p>

---
# Ressources 
* **Figure 1** : https://techtoday.lenovo.com/fr/fr/solutions/smb/hyperviseur
* **Figure 2 --> Figure 14** : Captures d'écran