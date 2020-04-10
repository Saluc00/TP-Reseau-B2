# TP Archi réseau : déploiement automatisé, git, sauvegarde de conf

*Par USEREAU Lucas*

## Sommaire

- [Mise en place du TP](##-mise-en-place-du-tp)
    - [Configuration réseau](###-configuration-réseau)
        - [Router R1](####-router-r1)
        - [Switch client-sw1](####-switch-client-sw1)
        - [Switch client-sw2](####-switch-client-sw2)
        - [Switch client-sw3](####-switch-client-sw3)
        - [Switch client-sw4](####-switch-client-sw4)
    - [Ping](###-ping)
        - [Admins](####-admins)
        - [Guests](####-guests)
        - [Infra](####-infra)
- [TP](##-TP)


## Mise en place du TP

### Configuration réseau

#### Router `R1`

```
interface FastEthernet1/0.10
 encapsulation dot1Q 10
 ip address 10.5.10.254 255.255.255.0
!
interface FastEthernet1/0.20
 encapsulation dot1Q 20
 ip address 10.5.20.254 255.255.255.0
!
interface FastEthernet1/1.30
 encapsulation dot1Q 30
 ip address 10.5.30.254 255.255.255.0
!
```

#### Switch client-sw1

```
interface Ethernet0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
 duplex auto
```

#### Switch client-sw2

```
interface Ethernet0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
 duplex auto
!
interface Ethernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 duplex auto
!
interface Ethernet0/2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 duplex auto
```

#### Switch client-sw3

```
interface Ethernet0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
 duplex auto
```

#### Switch client-sw4

```
interface Ethernet0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
 duplex auto
```


### Ping

#### Admins

*Admin1 to Admin3*

```
PC1> ping 10.5.10.13
84 bytes from 10.5.10.13 icmp_seq=1 ttl=64 time=0.434 ms
84 bytes from 10.5.10.13 icmp_seq=2 ttl=64 time=0.605 ms
84 bytes from 10.5.10.13 icmp_seq=3 ttl=64 time=0.570 ms
84 bytes from 10.5.10.13 icmp_seq=4 ttl=64 time=1.155 ms
84 bytes from 10.5.10.13 icmp_seq=5 ttl=64 time=0.614 ms
```

#### Guests

*Guests2 to guest3*

```
PC2> ping 10.5.20.13
84 bytes from 10.5.20.13 icmp_seq=1 ttl=64 time=0.434 ms
84 bytes from 10.5.20.13 icmp_seq=2 ttl=64 time=0.572 ms
84 bytes from 10.5.20.13 icmp_seq=3 ttl=64 time=0.614 ms
84 bytes from 10.5.20.13 icmp_seq=4 ttl=64 time=0.636 ms
84 bytes from 10.5.20.13 icmp_seq=5 ttl=64 time=0.534 ms
```

#### Infra

*Web to dns*

```
PC8> ping 10.5.30.11
84 bytes from 10.5.30.11 icmp_seq=1 ttl=64 time=0.183 ms
84 bytes from 10.5.30.11 icmp_seq=2 ttl=64 time=0.326 ms
84 bytes from 10.5.30.11 icmp_seq=3 ttl=64 time=0.259 ms
84 bytes from 10.5.30.11 icmp_seq=4 ttl=64 time=0.297 ms
84 bytes from 10.5.30.11 icmp_seq=5 ttl=64 time=0.309 ms
```

## TP