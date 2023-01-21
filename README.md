# Groupe de daniel_t 995143

# Projet Kubernetes Learning documentation
## introduction

#### Outils :
* **Windows 10 Professionnel** 
<!-- ## 1 - Schéma

![](img/Shema.png)

## 2 - Installation
**ATTENTION:** *Contrairement à ce qui est décrit sur le schéma, mes machines de test sont définies sur des IPs allant de `192.168.56.0` à `192.168.56.6`, car VirtualBox sur Windows nous y oblige à utiliser le `.56` et non le `.1` comme désiré.*
#### a) Machines virtuelles
Commencez par installer trois machines sous Debian 10 :
* Une gateway (*`GW01`*) avec une première carte réseau bridge et une deuxième configurée en host-only.
* Une manager (*`MG01`*) avec une carte réseau host-only.
* Une application (*`APP01`*) avec une carte réseau host-only.

*Notre machine cliente (*`CLI01`*) ne nous est pas très utile pour l'instant.*

#### b) Réseaux
##### Réseau hôte
Nous allons devoir créer notre réseau host-only et le configurer, pour cela allez dans l'onglé `Fichier > Gestionnaire de réseau hôte`. Puis nous allons configurer notre réseau de sorte à ce qu'il colle parfaitement avec notre schéma..<br>
Pour cela, nous avons choisi les configurations suivantes :
![](img/configHostNetworkManager.png)
##### Interfaces réseaux
Nos machine nécessite la configuration de leurs interfaces, pour ce faire, nous allons éditer le fichier `/etc/network/interface` sur nos 4 machines.
###### Gateway
Nous avons donc 2 cartes réseaux :
* `bridge`
```c
# The primary network interface, the dridge
allow-hotplug enp0s3
iface enp0s3 inet dhcp
```
* `host-only`
```c
# The secondary network interface, the host-only
allow-hotplug enp0s8
iface enp0s8 inet static
address 192.168.56.1
netmask 255.255.255.248
```
###### Others
La configuration est similaire pour les 3 autres machines à la différence de l'adresse IP qui est propre à chaque machine d'elle.<br>Par exemple pour la machine manager (*`MG01`*) nous avont :
```c
# The primary network interface, host-only for the manager
allow-hotplug enp0s3
iface enp0s3 inet static
address 192.168.56.2
netmask 255.255.255.248
gateway 192.168.56.1
```
## 3 - IPtables
#### a) Prés requis
Pour cette étape, nous aurons besoin d'installer IPtables bien sûr s'il n'existe pas sur votre machine, mais aussi iptables-persistent, un package permettant de sauvegarder nos configurations.
#### b) Configuration
Pour configurer les IPtables, nous allons commencer par permettre le forward en IPv4 via la commande :
```c
/sbin/sysctl -w net.ipv4.ip_forward=1
```
Puis nous allons utiliser la commande `MASQUERADE` pour permettre la connexion internet des autres machines en passant par notre gateway (*`GW01`*) :
```c
iptables -t nat -A POSTROUTING ! -d 192.168.56.1/29 -o enp0s3 -j MASQUERADE
```
Puis nous allons fermer notre serveur en commencent par laisser l'accès au port 22 pour notre connexion ssh via :
```c
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
```
Nous fermons en suite tous les entrés extérieurs :
```c
sudo iptables
-A INPUT -i enp0s3 -p tcp -m tcp --dport 22 -j ACCEPT
-A INPUT -i enp0s3 -p tcp -m tcp --dport 80 -j ACCEPT
-A INPUT -i lo -j ACCEPT
```

## 4 - DNS
#### a) Prés requis
Pour commencer, nous allons modifier notre hostname pour toutes les machines par le leur nom, par exemple pour la machine `manager` :
```bash
echo "manager" > /etc/hostname
```
Puis il nous faudra modifier notre fichier `/etc/resolv.conf ` comme suite :
```c
domain infrasys.local
search infrasys.local
nameserver 192.168.56.2
```
nous allons utiliser `Bind9` comme serveur DNS.
https://maximepiazzola.wordpress.com/2017/10/03/installer-et-configurer-un-serveur-dns-bind-9-sur-debian/
#### b) Configuration
Configurer tout simplement vos fichiers DNS dans le dossier `/etc/bind`.
Puis sur chaque machine configurée vos `resolv.conf` :
```c
nameserver 192.168.56.2
```

Voir [resolvconf](https://kifarunix.com/make-permanent-dns-changes-on-resolv-conf-in-linux/) pour rendre les changements des nameserver dans les /etc/resolv.conf persistent sur la gateway.
## 5 - DHCP
Nous allons implémenter notre serveur DHCP sur notre machine `manager` pour ce faire configuré de la sorte votre fichier DHCP, voir la [documentation official](https://wiki.debian.org/fr/DHCP_Server) :
```c
option domain-name "infrasys.local";
option domain-name-servers 192.168.56.2;

default-lease-time 600;
max-lease-time 7200;

ddns-update-style none;

authoritative;

subnet 192.168.56.0 netmask 255.255.255.248 {
  option routers                192.168.56.2;
  option domain-name-servers    infrasys.local;
  range                         192.168.56.5 192.168.56.6;
  host superlinux-wlan {
                hardware ethernet 08:00:27:2e:39:a3;
                fixed-address 192.168.56.3;
  }
}
```
En cas d'érreur, voir pour supprimer le fichier dans /run/dhcpd.pid
## 6 - SSH

Voici comment procéder pour configurer la connexion SSH sur vos serveurs :


Sur chaque serveur, assurez-vous d'avoir installé openssh-server. Si ce n'est pas le cas, exécutez la commande suivante :
```bash
sudo apt-get install openssh-server
```
#### a) Interdiction de connection par mot de passe

Sur chaque serveur, modifiez le fichier /etc/ssh/sshd_config en ajoutant les lignes suivantes :
```bash
PasswordAuthentication no
ChallengeResponseAuthentication no
UsePAM no
```

Cela désactivera l'authentification par mot de passe et l'authentification par réponse de défi, et empêchera l'utilisation de PAM (Pluggable Authentication Modules).
#### b) Banner

Pour afficher une bannière à la connexion, modifiez le fichier /etc/ssh/sshd_config sur chaque serveur en ajoutant la ligne suivante :
```bash
Banner /etc/ssh/banner
```
Créez le fichier /etc/ssh/banner et ajoutez votre bannière au format souhaité. Assurez-vous de remplacer {ServerName} et {login} par les valeurs appropriées.

#### c) Le PAT

Pour mettre en place le PAT sur GW01, vous pouvez utiliser iptables. Exécutez les commandes suivantes sur GW01 pour autoriser le trafic entrant sur les ports 2222 et 2223 et le rediriger vers les serveurs MG01 et APP01 respectivement :
```bash
sudo iptables -A INPUT -p tcp --dport 2222 -j ACCEPT
sudo iptables -A PREROUTING -t nat -p tcp --dport 2222 -j DNAT --to 192.168.56.2:22

sudo iptables -A INPUT -p tcp --dport 2223 -j ACCEPT
sudo iptables -A PREROUTING -t nat -p tcp --dport 2223 -j DNAT --to 192.168.56.3:22
```
Redémarrez le serveur SSH sur chaque serveur avec la commande suivante :
```bash
sudo service ssh restart
```
Vous devriez maintenant être en mesure de vous connecter à vos serveurs via SSH en utilisant une paire de clés privée/public, et d'accéder aux serveurs depuis l'extérieur via GW01 en utilisant les ports 2222 et 2223.


## Auteurs

Thomas DANIEL  [Linkedin](https://www.linkedin.com/in/thomas-daniel-607545203/)

## Version

* 1.0
    * Première version


