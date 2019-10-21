# TP2 : Network low-level, Switching

TP réalisé par *USEREAU Lucas* B2A.

# Sommaire

* [I. Simplest setup](#i-simplest-setup)
  * [Topologie](#topologie)
  * [Plan d'adressage](#plan-dadressage)
  * [ToDo](#todo)
* [II. More switches](#ii-more-switches)
  * [Topologie](#topologie-1)
  * [Plan d'adressage](#plan-dadressage-1)
  * [ToDo](#todo-1)
  * [Mise en évidence du Spanning Tree Protocol](#mise-en-évidence-du-spanning-tree-protocol)
* [III. Isolation](#iii-isolation)
  * [1. Simple](#1-simple)
    * [Topologie](#topologie-2)
    * [Plan d'adressage](#plan-dadressage-2)
    * [ToDo](#todo-2)
  * [2. Avec trunk](#2-avec-trunk)
    * [Topologie](#topologie-3)
    * [Plan d'adressage](#plan-dadressage-3)
    * [ToDo](#todo-3)
* [IV. Need perfs](#iv-need-perfs)
  * [Topologie](#topologie-4)
  * [Plan d'adressage](#plan-dadressage-4)
  * [ToDo](#todo-4)

# I. Simplest setup

## Topologie

```
+-----+        +-------+        +-----+
| PC1 +--------+  SW1  +--------+ PC2 |
+-----+        +-------+        +-----+
```

## Plan d'adressage

Machine | `net1`
--- | ---
`PC1` | `10.2.1.1/24`
`PC2` | `10.2.1.2/24`

## TODO

* Topologie OK !
* Ping ok !
```
PC-1> ping 10.2.1.2
84 bytes from 10.2.1.2 icmp_seq=1 ttl=64 time=0.000 ms
84 bytes from 10.2.1.2 icmp_seq=2 ttl=64 time=0.000 ms
84 bytes from 10.2.1.2 icmp_seq=3 ttl=64 time=0.000 ms
84 bytes from 10.2.1.2 icmp_seq=4 ttl=64 time=0.000 ms
84 bytes from 10.2.1.2 icmp_seq=5 ttl=64 time=0.000 ms
```
```
PC-2> ping 10.2.1.2
10.2.1.2 icmp_seq=1 ttl=64 time=0.001 ms
10.2.1.2 icmp_seq=2 ttl=64 time=0.001 ms
10.2.1.2 icmp_seq=3 ttl=64 time=0.001 ms
10.2.1.2 icmp_seq=4 ttl=64 time=0.001 ms
10.2.1.2 icmp_seq=5 ttl=64 time=0.001 ms
```

**Les PCs communiquent ensembles**

*En faisant une capture de tram sur le PC1*

En faisant un ping.

On vois qu'en premier le PC1 envoie une requete **ARP REQUEST**
```
source	10.2.1.1	destination 10.2.1.2	ICMP	98	Echo (ping) request  id=0x540c, seq=1/256, ttl=64 (reply in 16)```
```
et est repondu avec une requete **ARP REPLY**

```
source	10.2.1.2  destination	10.2.1.1	ICMP	98	Echo (ping) reply    id=0x540c, seq=1/256, ttl=64 (request in 15)
```

**Récapituler toutes les étapes (dans le compte-rendu, à l'écrit) quand PC1 exécute ping PC2 pour la première fois.**

Lors d'un *ping* de pc1 a pc2 pour la premiere fois, pc1 envoie en broadcast **QUI EST 10.2.1.2 ??**

```
source (pc1) Private_66:68:00	destination Broadcast	ARP	64	Who has 10.2.1.2? Tell 10.2.1.1
```
Le pc2 recois la demande et repond par :

```
source (pc2) Private_66:68:01	destination (pc1) Private_66:68:00	ARP	64	10.2.1.2 is at 00:50:79:66:68:01
```

s'en suis une suite de requete **ARP REQUEST** et **REPLY**, le fameux *ping* *(pong)*.

PC1 envoie une requete ARP REQUEST et PC2 repond par une requete ARP REPLY, le tout 5 fois.

- Le switch n'as pas besoin d'adresse IP car dans ce cas, il ne sert que de *multiprise*.

- Les machines, elles, ont besoins d'une IP car elle leurs servent d'adresse, pour se reconnaitre et se différencier des autres.

# II. More switches

## Topologie

```
                        +-----+
                        | PC2 |
                        +--+--+
                           |
                           |
                       +---+---+
                   +---+  SW2  +----+
                   |   +-------+    |
                   |                |
                   |                |
+-----+        +---+---+        +---+---+        +-----+
| PC1 +--------+  SW1  +--------+  SW3  +--------+ PC3 |
+-----+        +-------+        +-------+        +-----+
```

**OK !**

## Plan d'adressage

Machine | `net1`
--- | ---
`PC1` | `10.2.2.1/24`
`PC2` | `10.2.2.2/24`
`PC3` | `10.2.2.3/24`

## ToDo

* Topologie OK !
* Ping ok !

depuis PC3 je ping PC1 et PC2 (donc tout est ok pour tout les pc !):

```
PC-3> ping 10.2.2.1
10.2.2.1 icmp_seq=1 ttl=64 time=0.001 ms
10.2.2.1 icmp_seq=2 ttl=64 time=0.001 ms
10.2.2.1 icmp_seq=3 ttl=64 time=0.001 ms
10.2.2.1 icmp_seq=4 ttl=64 time=0.001 ms
10.2.2.1 icmp_seq=5 ttl=64 time=0.001 ms

PC-3> ping 10.2.2.2
84 bytes from 10.2.2.2 icmp_seq=1 ttl=64 time=0.810 ms
84 bytes from 10.2.2.2 icmp_seq=2 ttl=64 time=1.246 ms
```

* Analyser la table MAC d'un switch

```
IOU3# show mac address-table

          Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
   1    0050.7966.6802    DYNAMIC     Et0/2
   1    0050.7966.6803    DYNAMIC     Et0/1
   1    0050.7966.6804    DYNAMIC     Et0/1
   1    aabb.cc00.0210    DYNAMIC     Et0/1
   1    aabb.cc00.0400    DYNAMIC     Et0/1
   1    aabb.cc00.0410    DYNAMIC     Et0/0
Total Mac Addresses for this criterion: 6

```

Chaque ligne represente a quel Vlan appartient l'adresse mac et sur quelle port il est relié.

les 3 premieres sont (dans cette ordre) PC1, PC2 et PC3 suivie des 3 switch

- Il y a des trames CDP qui circulent. Quoi qu'est-ce ?

C'est Qu'est ce un protocole Cisco : `Cisco Discovery Protocol`. Toute les 60 secondes se protole est envoyé sur le réseau pour trouver d'autre périphérique voisin connecté.

## Mise en évidence du Spanning Tree Protocol

**Determiner les informations STP**

```
c'est un protocole réseau de niveau 2 permettant de déterminer une topologie réseau sans boucle 
```
- Qui est le root bridge, quels sont les ports désactivés, etc.

**IOU1**
```
VLAN0001
  Spanning tree enabled protocol rstp
  Root ID    Priority    32769
             Address     aabb.cc00.0200
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.0200
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Desg FWD 100       128.1    Shr
Et0/1               Desg FWD 100       128.2    Shr
Et0/2               Desg FWD 100       128.3    Shr
Et0/3               Desg FWD 100       128.4    Shr
Et1/0               Desg FWD 100       128.5    Shr
Et1/1               Desg FWD 100       128.6    Shr
Et1/2               Desg FWD 100       128.7    Shr
Et1/3               Desg FWD 100       128.8    Shr
```


**IOU2**
```

VLAN0001
  Spanning tree enabled protocol rstp
  Root ID    Priority    32769
             Address     aabb.cc00.0200
             Cost        100
             Port        2 (Ethernet0/1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.0300
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Desg FWD 100       128.1    Shr
Et0/1               Root FWD 100       128.2    Shr
Et0/2               Desg FWD 100       128.3    Shr
Et0/3               Desg FWD 100       128.4    Shr
Et1/0               Desg FWD 100       128.5    Shr
Et1/1               Desg FWD 100       128.6    Shr
Et1/2               Desg FWD 100       128.7    Shr
```


**IOU3**
```

VLAN0001
  Spanning tree enabled protocol rstp
  Root ID    Priority    32769
             Address     aabb.cc00.0200
             Cost        100
             Port        1 (Ethernet0/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.0400
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Root FWD 100       128.1    Shr
Et0/1               Altn BLK 100       128.2    Shr
Et0/2               Desg FWD 100       128.3    Shr
Et0/3               Desg FWD 100       128.4    Shr
Et1/0               Desg FWD 100       128.5    Shr
Et1/1               Desg FWD 100       128.6    Shr
Et1/2               Desg FWD 100       128.7    Shr
```

Qui est le root bridge ? `IOU1` ! 

Sur les `IOU2` le port root est **0/1** car il est directement relier au switch root 
Alors que `IOU3` a le port **0/0** car c'est lui qui est relier sur le switch root.
De plus, `IOU3` à desactivé le port **0/1** car il est relier au switch `IOU2` alors qu'il le vois deja a l'aide du port relié a `IOU1` (le switch ROOT) 

OK sous forme de tableau ça donne ça:


|    Switche    |    Role         |    Ports ouverts       |    Ports Fermés    |
|---------------|-----------------|------------------------|--------------------|
| **IOU1**     | `Route Bridge`  |`0/0`  `0/1` ¦ `0/2`      |     **`None`**      |
| **IOU2**      | `Passif`       |`0/0` ¦ `0/1 (vers root)` ¦ `0/2`      |     **`None`**      |
| **IOU3**      | `Passif`        |`0/0 (vers root)` ¦ **`0/1 (port coupé)`** ¦ `0/2` |       `0/1`         |

**Confirmer les informations STP**

Je fais le test en faisant un ping entre PC2 et PC3 !

|    Switches   |    ARP       |    ICMP    |
|---------------|-----------------|------------------------|--------------------|
| **IOU1 / IOU2** | `✔` | `✔` |
| **IOU1 / IOU3** | `✔` | `✔` |
| **IOU2 / IOU3** | `✔` | **`X`** |

**Déterminer quel lien a été désactivé par STP**

Le lien entre le switch IOU 2 et IOU 3 est désactiver !

**Faire un schéma qui explique le trajet d'une requête ARP lorsque PC1 ping PC3, et de sa réponse**

- Mon super schema !

![Mon super Schema !](https://gitlab.com/Saluc00/tp-reseau-b2/raw/master/2/schema.gif)

**Reconfigurer STP**

**Changer la priorité d'un switch qui n'est pas le root bridge**

Pour cela dans le terminal du switch 2:
```
conf
spanning-tree vlan 1 root primary
```

Le switch 2 est désormais root le switch 1 ne l'ai plus et le switch 3 n'a pas changé.

# III. Isolation

# 1. Simple

## Topologie

```
+-----+        +-------+        +-----+
| PC1 +--------+  SW1  +--------+ PC3 |
+-----+      10+-------+20      +-----+
                 20|
                   |
                +--+--+
                | PC2 |
                +-----+
```

**OK !**

## Plan d'adressage


Machine | IP `net1` | VLAN
--- | --- | ---
`PC1` | `10.2.3.1/24` | 10
`PC2` | `10.2.3.2/24` | 20
`PC3` | `10.2.3.3/24` | 20

## ToDo

**Mettre en place la topologie ci-dessus avec des VLANs**

Pour mettre les **VLANs** il me faire les manipulations suivante sur switch1:

*Pour configurer le VLAN 10 sur le port 0/0*
```
conf t
vlan 10
name client-network-10
exit
interface Ethernet 0/0
switchport mode access
switchport access vlan 10
```

*Pour configurer le VLAN 20 sur le port 0/1*
```
conf t
vlan 10
name client-network-20
exit
interface Ethernet 0/1
switchport mode access
switchport access vlan 20
```

*Pour configurer le VLAN 20 sur le port 0/2*
```
conf t
vlan 10
name client-network-20
exit
interface Ethernet 0/2
switchport mode access
switchport access vlan 20
```

**Faire communiquer les PCs deux à deux**

Ok.. à partir de maintenant, Je peut ping PC2 à PC3 (et vice versa) mais PC1 lui est tout seule !

**PC1**
```
3-pc1> ping 10.2.3.2
host (10.2.3.2) not reachable
```

**PC2**
```
3-pc2> ping 10.2.3.3
84 bytes from 10.2.3.3 icmp_seq=1 ttl=64 time=0.674 ms
84 bytes from 10.2.3.3 icmp_seq=2 ttl=64 time=0.976 ms
```

**PC3**
```
3-pc3> ping 10.2.3.2
84 bytes from 10.2.3.2 icmp_seq=1 ttl=64 time=0.659 ms
84 bytes from 10.2.3.2 icmp_seq=2 ttl=64 time=0.652 ms
```

# 2. Avec trunk

## Topologie

```
+-----+        +-------+        +-------+        +-----+
| PC1 +--------+  SW1  +--------+  SW2  +--------+ PC4 |
+-----+      10+-------+        +-------+20      +-----+
                 20|              10|
                   |                |
                +--+--+          +--+--+
                | PC2 |          | PC3 |
                +-----+          +-----+
```

*En faite se serra plus comme ça !*
```
          A        T           T       A
+----+    10+-----+10,20   10,20+-----+20      +-----+
|PC1+------+ SW1 +-------------+ SW2 +---------+ PC4 |
+----+      +-----+             +-----+        +-----+
              A|                   |A
             20|                   |10
               |                   |
               |                   |
            +--+--+             +--+--+
            | PC2 |             | PC3 |
            +-----+             +-----+
```

**OK !**

## Plan d'adressage

Machine | IP `net1` | IP `net2` | VLAN
--- | --- | --- | ---
`PC1` | `10.2.10.1/24` | X | 10
`PC2` | X | `10.2.20.1/24` | 20
`PC3` | `10.2.10.2/24` | X | 10
`PC4` | X | `10.2.20.2/24` | 20

## ToDo

**Mettre en place la topologie ci-dessus (en utilisant des VLAN, vous aurez besoin de la notion de trunk)**

Ok donc.. je parametre les ports des switchs en leurs assignant les VLAN 10 et 20 comme pour la partie simple *(basique !)*

**Seulement !** pour le ports des switches qui se relie !

soit, le port 0/2 de **SW1** et le port 0/0 de **SW2**, je doit activer le mode trunk

**SW1**
*Je n'autorise que les vlan 10 et 20 sur le trunk !*
```
interface Ethernet 0/2
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 10,20 
```

**SW2**
*Je n'autorise que les vlan 10 et 20 sur le trunk !*
```
interface Ethernet 0/0
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 10,20 
```

Et ça fonctionnneee !!!

Ping **PC1** à **PC3**
```
4-pc1> ping 10.2.10.2
84 bytes from 10.2.10.2 icmp_seq=1 ttl=64 time=0.592 ms
84 bytes from 10.2.10.2 icmp_seq=2 ttl=64 time=0.634 ms
```


Ping **PC2** à **PC4**
```

4-pc2> ping 10.2.20.2
84 bytes from 10.2.20.2 icmp_seq=1 ttl=64 time=0.945 ms
84 bytes from 10.2.20.2 icmp_seq=2 ttl=64 time=0.906 ms
```

**Mettre en évidence l'utilisation des VLANs avec Wireshark**

Ok donc la.. quand je fais des ping, en plus des requete ICMP, j'ai des requete **STP** comme cela 

```
source aa:bb:cc:00:06:20 destination PVST+	STP	68	RST. Root = 32768/10/aa:bb:cc:00:06:00  Cost = 0  Port = 0x8003
```

et dedans on peut y voir ceci :

```
Originating VLAN (PVID): 10
    Type: Originating VLAN (0x0000)
    Length: 2
    Originating VLAN: 10
```

Cette ligne `Originating VLAN: 10`, m'indique que la trame est passé par le **VLAN 10** !

# IV. Need perfs

Mise en place d'un agrégat de port afin de bénéficier d'une meilleure performance ainsi que d'une meilleure sécurité.

## Topologie

Pareil qu'en III.2. à part le lien entre SW1 et SW2 qui est doublé.

```
+-----+        +-------+--------+-------+        +-----+
| PC1 +--------+  SW1  |        |  SW2  +--------+ PC4 |
+-----+      10+-------+--------+-------+20      +-----+
                 20|              10|
                   |                |
                +--+--+          +--+--+
                | PC2 |          | PC3 |
                +-----+          +-----+

```
## Plan d'adressage

Pareil qu'en [III.2.](#2-avec-trunk).

Machine | IP `net1` | IP `net2` | VLAN
--- | --- | --- | ---
`PC1` | `10.2.10.1/24` | X | 10
`PC2` | X | `10.2.20.1/24` | 20
`PC3` | `10.2.10.2/24` | X | 10
`PC4` | X | `10.2.20.2/24` | 20

## ToDo

**Mettre en place la topologie ci-dessu**

Je dois mettre en place le fameux **LACP**

Pour cela je met un second lien entre **SW1** et **SW2**

***DONE !***

**Vérifier avec un show ip interface po1 que la bande passante a bien été doublée**

```
show ip interface po1
Port-channel1 is up, line protocol is up
```