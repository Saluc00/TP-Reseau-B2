# TP : Back to basics

TP réalisé par *USEREAU Lucas* B2A.

# Sommaire
* [0. Etapes préliminaires](#0-etapes-préliminaires)
* [I. Gather informations](#i-gather-informations)
* [II. Edit configuration](#ii-edit-configuration)
  * [1. Configuration cartes réseau](#1-configuration-cartes-réseau)
  * [2. Serveur SSH](#2-serveur-ssh)
* [III. Routage simple](#iii-routage-simple)
* [IV. Autres applications et métrologie](#iv-autres-applications-et-métrologie)
  * [1. Commandes](#1-commandes)
  * [2. Cockpit](#2-cockpit)
  * [3. Netdata](#3-netdata)



# 0. Etapes préliminaires

Du fais de la sortie de **l'iso Centos 08**, j'ai du initié une nouvelle machine virtuel.

* Elle possede une **carte NAT** qui lui permet d'avoir indernet.

* Une autre carte réseau qui me permet de joindre la vm par ssh.

## I. Gather informations

### Récupérer une liste des cartes réseau avec leur nom, leur IP et leur adresse MAC ?

    * **ip a**
```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:77:b8:66 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic noprefixroute enp0s3
       valid_lft 85666sec preferred_lft 85666sec
    inet6 fe80::a00:27ff:fe77:b866/64 scope link
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:68:9f:98 brd ff:ff:ff:ff:ff:ff
    inet 192.168.10.3/24 brd 192.168.10.255 scope global dynamic noprefixroute enp0s8
       valid_lft 324sec preferred_lft 324sec
    inet6 fe80::a765:6bc4:f184:d934/64 scope link noprefixroute
       valid_lft forever preferred_lft forever

```

### Déterminer si les cartes réseaux ont récupéré une IP en DHCP ou non
    * Pour **enp0s3** : `cat /etc/sysconfig/network-scripts/ifcfg-enp0s3 | grep BOOTPROTO`

    ```
     BOOTPROTO=dhcp
    ```
    * Pour **enp0s8** : `cat /etc/sysconfig/network-scripts/ifcfg-enp0s8 | grep BOOTPROTO`
    ```
     BOOTPROTO=dhcp
    ```

*  Afficher la table de routage de la machine et sa table ARP

**Table de routage**

```
default via 10.0.2.2 dev enp0s3 proto dhcp metric 101
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 101
192.168.10.0/24 dev enp0s8 proto kernel scope link src 192.168.10.3 metric 100

```

**Table ARP**

```
Adresse                  TypeMap AdresseMat          Indicateurs           Iface
192.168.10.1             ether   0a:00:27:00:00:18   C                     enp0s8
_gateway                 ether   52:54:00:12:35:02   C                     enp0s3
192.168.10.2             ether   08:00:27:3d:a8:f2   C                     enp0s8

```

### Récupérer la liste des ports en écoute (listening) sur la machine (TCP et UDP)

```
Netid  State    Recv-Q   Send-Q              Local Address:Port       Peer Address:Port
udp    UNCONN   0        0                       127.0.0.1:323             0.0.0.0:*       users:(("chronyd",pid=732,fd=6))
udp    UNCONN   0        0             192.168.10.3%enp0s8:68              0.0.0.0:*       users:(("NetworkManager",pid=767,fd=19))
udp    UNCONN   0        0                10.0.2.15%enp0s3:68              0.0.0.0:*       users:(("NetworkManager",pid=767,fd=22))
udp    UNCONN   0        0                           [::1]:323                [::]:*       users:(("chronyd",pid=732,fd=7))
tcp    LISTEN   0        128                       0.0.0.0:22              0.0.0.0:*       users:(("sshd",pid=6816,fd=6))
tcp    LISTEN   0        128                          [::]:22                 [::]:*       users:(("sshd",pid=6816,fd=8))

```
On peut y voir les port ouvert **323, 68, 22**.

### Afficher l'état actuel du firewall

Les interfaces filtré sont `interfaces: enp0s8 enp0s3`. 

# II. Edit configuration

## 1. Configuration cartes réseau

### Modifier la configuration de la carte réseau privée

`vi /etc/sysconfig/network-scripts/ifcfg-enp0s8`

Parametrer :

```
BOOTPROTO=static
NAME=enp0s8
DEVICE=enp0s8
ONBOOT=yes

IPADDR=192.168.10.10
NETMASK=255.255.255.0
GATEWAY=192.168.10.1
```

puis `sudo nmcli con reload` pour actualiser les parametres.

### Mettre en place un NIC teaming

*(je savais du tout ce que c'etais..)*

Pour ça j'ai procéder de cette maniere :
* j'ai activé le bonding : ` modprobe bonding`

* créer une nouvelle interface : `vi /etc/sysconfig/network-scripts/ifcfg-bond0`

* initialisé la conf suivante :

```
DEVICE=bond0
DEVICE=bond0
BONDING_OPTS="miimon=1 updelay=0 downdelay=0 mode=active-backup" TYPE=Bond
BONDING_MASTER=yes
BOOTPROTO=none
IPADDR=192.168.10.5
NETMASK=255.255.255.0
PREFIX=24
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=bond0
UUID=bbe539aa-5042-4d28-a0e6-2a4d4f5dd744
ONBOOT=yes
```

* ajouté deux ligne au configuration des cartes `enp0s8` et `enp0s9`.

```
MASTER=bond0
SLAVE=yes
```

* Puis `nmcli con reload` et le tour est joué !

je me trouve alors avec toutes ces cartes la :

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:d7:c8:6f brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic noprefixroute enp0s3
       valid_lft 85727sec preferred_lft 85727sec
    inet6 fe80::a00:27ff:fed7:c86f/64 scope link
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:78:a2:3c brd ff:ff:ff:ff:ff:ff
    inet 192.168.10.10/24 brd 192.168.10.255 scope global noprefixroute enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe78:a23c/64 scope link
       valid_lft forever preferred_lft forever
4: enp0s9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:1b:0d:89 brd ff:ff:ff:ff:ff:ff
    inet 192.168.10.20/24 brd 192.168.10.255 scope global noprefixroute enp0s9
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe1b:d89/64 scope link
       valid_lft forever preferred_lft forever
5: bond0: <NO-CARRIER,BROADCAST,MULTICAST,MASTER,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
    link/ether 5e:9f:ea:3b:bc:ff brd ff:ff:ff:ff:ff:ff
    inet 192.168.2.12/24 brd 192.168.2.255 scope global noprefixroute bond0
       valid_lft forever preferred_lft forever
```

## 2. Serveur SSH

###  Modifier la configuration du système pour que le serveur SSH tourne sur le port 2222

* dans `vi /etc/ssh/sshd_config` mettre ` Port 2222`

* `semanage port -a -t ssh_port_t -p tcp 2222`

* `rewall-cmd --permanent --zone=public --add-port=2222/tcp`

* `firewall-cmd --reload`

* `systemctl restart sshd.service`

* avec `ss -tnlp | grep ssh`:

```
LISTEN   0         128                 0.0.0.0:2222             0.0.0.0:*        users:(("sshd",pid=782,fd=6))          
LISTEN   0         128                    [::]:2222                [::]:*        users:(("sshd",pid=782,fd=8))
```

Maintenant pour me co en **ssh**, je doit faire `ssh root@192.168.10.5 -p 2222`

# III. Routage Simple

* Vm1 
    * ip : 192.168.1.10
    * masque : 255.255.255.0
    * gateway : 192.168.1.1

* Vm2
    * ip : 192.168.2.1010
    * masque : 255.255.255.0
    * gateway : 192.168.2.1

* Router
    * ip : 192.168.1.1 et 192.168.2.1 et une en **NAT**
    * masque : 255.255.255.0

sur la VM router faire 

```
SELINUX=permissive

iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE

iptables-save > /etc/sysconfig/iptables
``` 

**les ping sur les vm 1 et 2 son OK vers l'extérieur !**

# IV. Autres applications et métrologie

## 1. Commandes

Iftop CKOI ?

En faite, c'est un wireshark en ligne de commande qui permet de voir en temps réel les communication de la carte selectionné !

## 2. Cockpit

Cockpit OK !

## 3. Netdata

Netdata OK !
