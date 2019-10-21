# TP3 : Routage INTER-VLAN + mise en situation

# Sommaire

* [I. *Router-on-a-stick*](#i-router-on-a-stick)
* [II. Cas concret](#ii-cas-concret)

**Petit détails..**

- Les `PC` sont des **VPCS** de GNS3 (sauf indication contraire)
- Les `P` sont des **imprimantes**, on les simulera avec des **VPCS** aussi
- Les `SRV` sont des **serveurs**, **VPCS** again
- Les `SW` sont des **Switches** Cisco, virtualisé avec l'**IOU L2**
- Les `R` sont des **routeurs**, virtualisé avec l'iOS dispo ici : **Cisco 3640**

# I. Router-on-a-stick

**Schema**

```
             +--+
             |R1|
             +-++
               |
               |                    +---+
               |          +---------+PC4|
+---+        +-+-+      +---+       +---+
|PC1+--------+SW1+------+SW2|
+---+        +-+-+      +-+--+
               |          |  |
               |          |  +------+--+
               |          |         |P1|
             +-+-+      +-+-+       +--+
             |PC2|      |PC3|
             +---+      +---+
```

**Tableau des réseaux utilisés**

Réseau | Adresse | VLAN | Description
--- | --- | --- | ---
`net1` | `10.3.10.0/24` | 10 | Utilisateurs
`net2` | `10.3.20.0/24` | 20 | Admins
`net3` | `10.3.30.0/24` | 30 | Visiteurs
`netP` | `10.3.40.0/24` | 40 | Imprimantes

**Tableau d'adressage**

Machine | VLAN | IP `net1` | IP `net2` | IP `net3` |  IP `netP`
--- | --- | --- | --- | --- | ---
PC1 | 10 | `10.3.10.1/24` | x | x | x
PC2 | 20 | x | `10.3.20.2/24` | x | x | x
PC3 | 20 | x | `10.3.20.3/24` | x | x | x
PC4 | 30 | x | x |  `10.3.30.4/24` | x | x
P1 | 40 | x | x | x | `10.3.40.1/24` 
R1 | x |  `10.3.10.254/24` | `10.3.20.254/24` | `10.3.30.254/24` | `10.3.40.254/24` 

**Qui peut joindre qui ?**

✅ = peuvent se joindre
❌ = ne peuvent pas se joindre

Réseaux | `net1` |  `net2` |  `net3` |  `netP`
--- | --- | --- | --- | ---
 `net1` | ✅ | ❌ | ❌ | ❌
 `net2` | ❌ | ✅ | ✅ | ✅
 `net3` | ❌ | ✅ | ✅ | ✅

**Setup this shit**

Ok ! ça c'est rapide !

**You'll need inter-VLAN routing to make it work properly**

La c'est déja plus long..

On a 2 switches ! Il me faut mettre les 4 **VLAN** sur les 2 switch et mettre leurs interfaces commune en **mode trunk** !

Pour les Vlan sur les switch faire :
*Ca c'est la création des VLANs*
```
conf t
vlan {{Numéro du VLAN}}
name {{Nom du VLAN}}
exit
```

*Pour assigner un VLAN à un port*
```
conf t
interface ethernet {{port}}
switchport mode access
switchport access vlan {{Numéro du VLAN}}
```

*Et enfin pour activer le mode trunk*
```
conf t
interface ethernet {{port}}
switchport trunk encapsulation dot1q
switchport mode trunk

/* 
 * J'ai authoriser manuellement tel et tel VLAN
 * pck je n'ai pas trouver comment dire que j'accepte tout les VLAN 
 */
switchport trunk allowed vlan 10,20,30,40 
```

***Ca c'est pour les switches du plaisir***

Maintenant GO le routeur !

Je souhaite attribuer des **sous-interfaces**.

```
conf t
interface fastethernet 0/0
ip address 10.3.0.1 255.255.255.0
no shut
/*
 * Dans le doute je paramatre l'interface physique en premier
 */

exit
interface fastethernet 0/0.{{VLAN}}
encapsulation dot1q {{VLAN}}
ip address 10.3.{{VLAN}}.254 255.255.255.0
exit

// Puis recommencer autant de fois pour tout les Vlan que lon souhaite !
```

Maintenant que tout est configuré, tout les **PC** ping **R1** !

**PC1**
```
PC-1> ping 10.3.10.254
84 bytes from 10.3.10.254 icmp_seq=1 ttl=255 time=8.655 ms
```

**PC2**
```
PC-2> ping 10.3.20.254
84 bytes from 10.3.20.254 icmp_seq=1 ttl=255 time=7.833 ms
```

**PC3**
```
PC-3> ping 10.3.20.254
84 bytes from 10.3.20.254 icmp_seq=1 ttl=255 time=8.609 ms
```

**PC4**
```
PC-4> ping 10.3.30.254
84 bytes from 10.3.30.254 icmp_seq=1 ttl=255 time=7.839 ms
```

**P1**
```
p1> ping 10.3.40.254
84 bytes from 10.3.40.254 icmp_seq=1 ttl=255 time=8.649 ms
```
**Prove me that your setup is actually working**

**Fé** juste au dessus 🌞

# II. Cas concret

Il faut que je fasse ça..........

![Yo](https://gitlab.com/Saluc00/b2-reseau-2019/raw/master/tp/3/pics/schema-II.png)

* `R1` `R3` `R4` et `R5` sont des bureaux avec des utilisateurs
* `R2` est une salle serveur 
* le bâtiment a une taille de 20m x 20m (approximativement, vous en aurez besoin sur la fin)

**C'est quoi ces machines ?**

Type | Nom | Rôle | Dans GNS 
--- | --- | --- | ---
`A` | Admins | Accès à tout à frer. Full power. | VPCS
`U` | Users | Accès à un peu moins. | VPCS
`S` | Stagiaires | Encore un peu moins. | VPCS
`SRV` | Serveurs | Services hébergés en local. Ceux encadrés en rouge sont des **serveurs sensibles ou SS** | VPCS (ou autre si explicitement demandé)
`P` | Imprimantes | Imprimantes dispo en réseau

**Qui a accès à qui exactement ?**

✅ = peuvent se joindre
❌ = ne peuvent pas se joindre

X | Admins | Users | Stagiaires | Serveurs | SS | Imprimantes
--- | --- | --- | --- | --- | --- | --- | 
Admins | ✅ | ❌ | ❌ | ✅ | ✅ | ✅ |
Users | ❌ | ✅ | ❌ | ✅ | ❌ | ✅ |
Stagiaires | ❌ | ❌ | ✅ | ❌ | ❌ | ✅ |
Serveurs | ✅ | ✅ | ❌ | ✅ | ❌ | ✅ |
Serveurs sensibles | ✅ | ❌ | ❌ | ❌ | ✅ | ❌ |
Imprimantes | ✅ | ✅ | ✅ | ✅ | ❌ | ✅ |
