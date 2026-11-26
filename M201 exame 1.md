# 📘 CC N°01 — Guide détaillé de révision Réseaux

> 🎯 **Objectif :** comprendre les concepts, savoir expliquer leur fonctionnement et reconnaître les commandes Cisco importantes dans Packet Tracer.

## 📋 Modules du CC

1. Configuration de base des périphériques
2. 🛠️ Concepts de commutation
3. 🔀 VLANs
4. 🏷️ Routage Inter-VLAN
5. 🔁 Concepts du STP
6. 🌳 EtherChannel
7. 🔗 DHCPv4
8. 📡 SLAAC et DHCPv6
9. 🌍 FHRP — First Hop Redundancy Protocol

---

# 1️⃣ Module 1 — Configuration de base des périphériques

## Utilise les commandes `show` pour vérifier VLAN, trunk, HSRP, DHCP, CDP et STP selon les demandes du sujet.

| Vérification               | Commande                     |
| -------------------------- | ---------------------------- |
| VLAN                       | `show vlan brief`            |
| Trunk                      | `show interfaces trunk`      |
| HSRP                       | `show standby brief`         |
| DHCP — adresses attribuées | `show ip dhcp binding`       |
| DHCP — pools               | `show ip dhcp pool`          |
| CDP — voisins              | `show cdp neighbors`         |
| CDP — détails              | `show cdp neighbors detail`  |
| STP                        | `show spanning-tree`         |
| STP pour le VLAN 20        | `show spanning-tree vlan 20` |

## 🎛️ 1.1 Les modes Cisco IOS

Cisco IOS utilise plusieurs modes.

### 👤 Mode utilisateur

```text
Switch>
```

Utilisé pour quelques commandes de consultation.

Pour passer au mode privilégié :

```bash
enable
```

### 👑 Mode privilégié

```text
Switch#
```

On peut utiliser les commandes `show` et entrer dans la configuration.

```bash
show running-config
```

### ⚙️ Mode configuration globale

```bash
Switch(config)#
```

Commande :

```bash
configure terminal
```

### 🔌 Mode interface

```bash
Switch(config-if)#
```

Commande :

```bash
interface gigabitEthernet 0/1
```

### 🧠 À mémoriser

```text
Switch>       → User EXEC
Switch#       → Privileged EXEC
Switch(config)# → Global configuration
Switch(config-if)# → Interface configuration
```

---

## 🏷️ 1.2 Configurer le nom du switch

```bash
enable
configure terminal
hostname SW1
```

Le prompt devient :

```text
SW1(config)#
```

### 🤔 Pourquoi ?

Le hostname permet d'identifier facilement chaque équipement.

---

## 🚫 1.3 Désactiver la résolution DNS

```bash
no ip domain-lookup
```

### 🤔 Pourquoi ?

Si tu écris accidentellement une commande incorrecte, le switch peut essayer de résoudre cette commande comme un nom DNS.

Cette commande évite cette attente.

---

## 🔑 1.4 Configurer un mot de passe privilégié

```bash
enable secret class
```

`enable secret` protège l'accès au mode privilégié.

---

## 🖥️ 1.5 Configurer la console

```bash
line console 0
password cisco
login
```

### 💬 Explication

- `line console 0` → configuration de la console
- `password cisco` → définit le mot de passe
- `login` → demande le mot de passe

---

## 🔐 1.6 Configurer SSH

SSH permet d'administrer le périphérique à distance de manière sécurisée.

Exemple :

```bash
ip domain-name entreprise.local
username admin secret cisco123
crypto key generate rsa
line vty 0 4
login local
transport input ssh
```

### 💬 Explication

```text
username admin secret cisco123
```

→ crée un utilisateur local.

```text
crypto key generate rsa
```

→ crée les clés nécessaires à SSH.

```text
login local
```

→ utilise la base d'utilisateurs locale.

```text
transport input ssh
```

→ autorise SSH.

---

## 🌐 1.7 Configurer une adresse IP sur un switch

Un switch Layer 2 peut recevoir une adresse IP de gestion sur une interface VLAN.

```bash
interface vlan 1
ip address 192.168.1.2 255.255.255.0
no shutdown
```

Passerelle :

```bash
ip default-gateway 192.168.1.1
```

### ⚠️ Important

L'adresse IP de gestion du switch n'est pas une adresse IP attribuée à chaque port physique.

Elle est configurée sur une **SVI (Switch Virtual Interface)**.

---

## 🔍 1.8 Vérification

```bash
show running-config
show startup-config
show ip interface brief
show version
```

### Différence importante

```text
running-config → configuration actuellement en RAM
startup-config → configuration sauvegardée pour le démarrage
```

Pour sauvegarder :

```bash
copy running-config startup-config
```

---

# 2️⃣ Module 2 — Concepts de commutation

# 🔀 2.1 Qu'est-ce qu'un switch ?

Un switch est un équipement de couche 2 qui utilise principalement les **adresses MAC** pour transmettre les trames Ethernet.

Exemple :

```text
PC1 ---- Fa0/1
          |
        Switch
          |
        Fa0/2 ---- PC2
```

Le switch apprend quelle adresse MAC se trouve sur quel port.

---

# 🪪 2.2 Adresse MAC

Une adresse MAC identifie une interface réseau au niveau Ethernet.

Exemple :

```text
00:11:22:33:44:55
```

Le switch utilise ces adresses dans sa table MAC.

---

# 📚 2.3 Comment le switch apprend-il ?

Supposons :

```text
PC1 → Switch Fa0/1
MAC PC1 = AAAA.AAAA.AAAA
```

PC1 envoie une trame.

Le switch regarde l'adresse MAC **source** et apprend :

```text
AAAA.AAAA.AAAA → Fa0/1
```

### 🧠 Règle

**Le switch apprend l'adresse MAC source.**

---

# 📤 2.4 Forwarding

Si le switch connaît la MAC destination :

```text
MAC destination → Fa0/2
```

Il envoie la trame uniquement sur Fa0/2.

Cela s'appelle le **forwarding**.

---

# 🌊 2.5 Flooding

Si la MAC destination est inconnue, le switch diffuse la trame sur les ports appropriés.

C'est le **flooding**.

Exception :

🚫 Il ne renvoie pas la trame par le port sur lequel elle est arrivée.

---

# 📢 2.6 Broadcast

Une trame broadcast est destinée à tous les équipements du domaine de broadcast.

Adresse MAC Ethernet broadcast :

```text
FF:FF:FF:FF:FF:FF
```

Un switch diffuse le broadcast dans le VLAN concerné.

---

# 💥 2.7 Domaine de collision

Avec un switch :

```text
PC1 ── Port 1
PC2 ── Port 2
PC3 ── Port 3
```

Chaque port constitue normalement un domaine de collision séparé.

### 📌 À retenir

```text
Switch → réduit les collisions
VLAN → sépare les domaines de broadcast
```

---

# ⚖️ 2.8 Store-and-forward vs Cut-through

## 📦 Store-and-forward

Le switch reçoit toute la trame avant de la transmettre.

Il peut vérifier la trame avant de l'envoyer.

### ✅ Avantage

Plus de contrôle sur les erreurs.

### ⚠️ Inconvénient

Un peu plus de latence.

## 🏃 Cut-through

Le switch commence à transmettre avant de recevoir toute la trame.

### ✅ Avantage

Faible latence.

### ⚠️ Inconvénient

Il peut transmettre une trame incorrecte avant de détecter certains problèmes.

---

# 3️⃣ Module 3 — VLAN

# 📖 3.1 Définition

**VLAN = Virtual Local Area Network**

Un VLAN divise logiquement un réseau physique.

Exemple :

```text
Switch
├── VLAN 10 → Administration
├── VLAN 20 → Étudiants
└── VLAN 30 → Serveurs
```

Même switch physique, mais plusieurs réseaux logiques.

---

# 🤔 3.2 Pourquoi utiliser des VLAN ?

### 🔒 1. Sécurité

Les utilisateurs de VLAN différents sont séparés au niveau Layer 2.

### 📉 2. Réduction du broadcast

Chaque VLAN constitue son propre domaine de broadcast.

### 🗂️ 3. Organisation

On peut séparer :

```text
Administration
Informatique
Invités
Serveurs
```

---

# 🆕 3.3 Créer un VLAN

```bash
enable
configure terminal
vlan 10
name ADMIN
```

Créer VLAN 20 :

```bash
vlan 20
name ETUDIANTS
```

---

# 🚪 3.4 Configurer un port Access

Supposons que PC1 soit dans VLAN 10.

```bash
interface fa0/1
switchport mode access
switchport access vlan 10
```

### Que signifie Access ?

Un port access appartient normalement à un seul VLAN.

```text
PC1
 |
Fa0/1
 |
Switch
VLAN 10
```

---

# 🚦 3.5 Port Trunk

Un trunk permet de transporter plusieurs VLAN entre des équipements réseau.

Exemple :

```text
Switch 1                 Switch 2
VLAN 10 ─┐             ┌─ VLAN 10
VLAN 20 ─┼── Trunk ────┼─ VLAN 20
VLAN 30 ─┘             └─ VLAN 30
```

Configuration :

```bash
interface g0/1
switchport mode trunk
```

### 🧠 À retenir

```text
Access → généralement 1 VLAN
Trunk → plusieurs VLAN
```

---

# 🏷️ 3.6 802.1Q

Sur un trunk, le protocole IEEE 802.1Q permet d'identifier le VLAN relei à une trame.

Le VLAN est indiqué par un tag dans la trame Ethernet.

---

# 🧭 3.7 Native VLAN

Sur un trunk, le **Native VLAN** est utilisé pour les trames non taguées.

Il faut que les deux côtés du trunk aient une configuration cohérente.

---

# 🔍 3.8 Vérification VLAN

```bash
show vlan brief
```

Pour les trunks :

```bash
show interfaces trunk
```

---

# ⚠️ Piège d'examen

Si deux PC sont dans des VLAN différents :

```text
PC1 → VLAN 10
PC2 → VLAN 20
```

Ils ne communiquent pas directement au niveau Layer 2.

Il faut du **routage Inter-VLAN**.

---

# 4️⃣ Module 4 — Routage Inter-VLAN

# 🤔 4.1 Pourquoi le routage Inter-VLAN ?

Un VLAN est un domaine de broadcast séparé.

Pour permettre :

```text
VLAN 10 ↔ VLAN 20
```

il faut un équipement de couche 3.

---

# 🛣️ 4.2 Router-on-a-Stick

Router-on-a-Stick utilise :

- un routeur
- un lien trunk
- plusieurs sous-interfaces

Topologie :

```text
PC VLAN 10
     |
   Switch
     |
   TRUNK
     |
   Router
```

---

# ⚙️ 4.3 Configuration

Interface physique :

```bash
interface g0/0
no shutdown
```

Sous-interface VLAN 10 :

```bash
interface g0/0.10
encapsulation dot1Q 10
ip address 192.168.10.1 255.255.255.0
```

Sous-interface VLAN 20 :

```bash
interface g0/0.20
encapsulation dot1Q 20
ip address 192.168.20.1 255.255.255.0
```

---

# 🚪 4.4 Passerelle par défaut

PC du VLAN 10 :

```text
IP      : 192.168.10.10
Mask    : 255.255.255.0
Gateway : 192.168.10.1
```

PC du VLAN 20 :

```text
IP      : 192.168.20.10
Mask    : 255.255.255.0
Gateway : 192.168.20.1
```

### 🧠 Règle

La passerelle permet au PC d'envoyer des paquets vers un autre réseau.

---

# 🚀 4.5 SVI sur un switch Layer 3

Un switch Layer 3 peut faire du routage Inter-VLAN avec des SVI.

Exemple :

```bash
interface vlan 10
ip address 192.168.10.1 255.255.255.0
no shutdown

interface vlan 20
ip address 192.168.20.1 255.255.255.0
no shutdown
```

Puis :

```bash
ip routing
```

---

# ⚠️ Piège d'examen

Pour Router-on-a-Stick, il faut généralement :

```text
PC → port access
Switch → port trunk
Router → sous-interfaces dot1Q
```

Si le trunk est mal configuré, le routage Inter-VLAN ne fonctionnera pas.

---

# 5️⃣ Module 5 — STP

# 🌳 5.1 Pourquoi STP ?

Imagine :

```text
Switch A ─── Switch B
    │           │
    └───────────┘
```

Il existe une boucle Layer 2.

Les trames peuvent circuler continuellement.

Cela peut provoquer :

- Broadcast storm
- Instabilité de la table MAC
- Saturation du réseau

**STP = Spanning Tree Protocol**

STP évite ces boucles.

---

# 👑 5.2 Root Bridge

STP élit un switch comme :

**Root Bridge**

Le Root Bridge sert de référence pour calculer les chemins.

---

# 🆔 5.3 Bridge ID

Le Bridge ID est principalement composé de :

```text
Priority + MAC Address
```

Le switch avec le **plus petit Bridge ID** devient Root Bridge.

### ❗ Important

Pour influencer le choix :

➡️ On peut réduire la priorité.

Exemple :

```bash
spanning-tree vlan 10 priority 4096
```

---

# 🎯 5.4 Root Port

Sur un switch non-root :

**Root Port = meilleur chemin vers le Root Bridge.**

Un switch non-root possède normalement un Root Port par VLAN/instance STP.

---

# 📌 5.5 Designated Port

Le **Designated Port** est le port sélectionné pour transmettre sur un segment.

---

# ⛔ 5.6 Port bloqué / Alternate

Un chemin redondant peut être bloqué afin d'empêcher la boucle.

Si le chemin actif tombe, STP peut permettre l'utilisation du chemin de secours selon le fonctionnement de la version STP utilisée.

---

# 🔍 5.7 Vérification

```bash
show spanning-tree
```

Cette commande permet notamment d'identifier :

- Root Bridge
- Root Port
- Designated Ports
- États des ports

---

# 🔗 5.8 STP et EtherChannel

STP voit un EtherChannel comme un **seul lien logique**.

Cela permet d'utiliser plusieurs liens physiques sans que STP considère chaque câble comme une boucle indépendante.

---

# 6️⃣ Module 6 — EtherChannel

# 📖 6.1 Définition

EtherChannel regroupe plusieurs liens physiques en un seul lien logique.

Exemple :

```text
Switch A                 Switch B

Gi0/1 ───────────────── Gi0/1
Gi0/2 ───────────────── Gi0/2
Gi0/3 ───────────────── Gi0/3

       ↓

    Port-Channel 1
```

---

# 🤔 6.2 Pourquoi EtherChannel ?

### 📈 Bande passante

Plusieurs liens peuvent être utilisés comme un groupe.

### 🛡️ Redondance

Si un lien physique tombe, le groupe peut continuer avec les autres liens.

### 🌳 STP

STP considère le Port-Channel comme un seul lien logique.

---

# 🤝 6.3 LACP

**LACP = Link Aggregation Control Protocol**

C'est un standard IEEE.

Modes :

```text
active
passive
```

### 🟢 Active

Le périphérique tente activement de former l'EtherChannel.

### 🟡 Passive

Le périphérique attend des paquets LACP.

### ❗ Important

```text
active + active → fonctionne
active + passive → fonctionne
passive + passive → ne forme pas le groupe
```

---

# 🔷 6.4 PAgP

PAgP est un protocole Cisco.

Modes :

```text
desirable
auto
```

```text
desirable + desirable → fonctionne
desirable + auto → fonctionne
auto + auto → ne forme pas le groupe
```

---

# 💡 6.5 Exemple LACP

```bash
interface range g0/1-2
channel-group 1 mode active
```

Puis :

```bash
interface port-channel 1
```

Vérification :

```bash
show etherchannel summary
```

---

# ⚠️ Piège d'examen

Les interfaces qui forment un EtherChannel doivent avoir une configuration compatible.

Par exemple, les paramètres importants doivent être cohérents :

- vitesse
- duplex
- mode access/trunk
- VLAN
- paramètres trunk

---

# 7️⃣ Module 7 — DHCPv4

# 🤔 7.1 Pourquoi DHCP ?

Sans DHCP, chaque PC doit être configuré manuellement.

DHCP permet de distribuer automatiquement :

- Adresse IPv4
- Masque
- Passerelle
- DNS
- autres paramètres

---

# 🎬 7.2 DORA

Le processus DHCP classique est :

```text
D → Discover
O → Offer
R → Request
A → Acknowledgment
```

## 🔎 1. Discover

Le client cherche un serveur DHCP.

```text
Client → DHCP Discover
```

## 🎁 2. Offer

Le serveur propose une configuration.

```text
DHCP Server → DHCP Offer
```

## 🙋 3. Request

Le client demande l'offre.

```text
Client → DHCP Request
```

## ✅ 4. ACK

Le serveur confirme.

```text
DHCP Server → DHCP ACK
```

### 🧠 À mémoriser

```text
DORA = Discover → Offer → Request → ACK
```

---

# 🖧 7.3 DHCP Cisco

Exemple :

```bash
ip dhcp excluded-address 192.168.1.1 192.168.1.10

ip dhcp pool LAN
network 192.168.1.0 255.255.255.0
default-router 192.168.1.1
dns-server 8.8.8.8
```

## 💬 Explication

```text
ip dhcp excluded-address
```

→ exclut certaines adresses du pool.

```text
ip dhcp pool LAN
```

→ crée le pool DHCP.

```text
network
```

→ définit le réseau distribué.

```text
default-router
```

→ indique la passerelle aux clients.

```text
dns-server
```

→ indique le serveur DNS.

---

# 🔍 7.4 Vérification

```bash
show ip dhcp binding
show ip dhcp pool
```

---

# 🔁 7.5 DHCP Relay

Si le serveur DHCP est dans un autre réseau, le routeur peut relayer les requêtes DHCP.

Commande :

```bash
interface g0/0
ip helper-address 192.168.100.10
```

### 🧠 À retenir

```text
Client → Routeur → DHCP Server
```

`ip helper-address` permet de relayer certains broadcasts de services, notamment DHCP.

---

# 8️⃣ Module 8 — SLAAC et DHCPv6

# 🤔 8.1 Pourquoi IPv6 automatique ?

Comme IPv4, IPv6 peut être configuré manuellement ou automatiquement.

Les mécanismes importants sont :

- SLAAC
- Stateless DHCPv6
- Stateful DHCPv6

---

# 🪄 8.2 SLAAC

**SLAAC = Stateless Address Autoconfiguration**

Le routeur envoie un message :

**RA = Router Advertisement**

Le PC utilise les informations du RA pour construire/configurer son adresse IPv6.

---

# HERE 

# 📡 8.3 RS et RA

### 🙋 RS

**Router Solicitation**

Le client demande des informations aux routeurs.

```text
Client → RS → Router
```

### 📣 RA

**Router Advertisement**

Le routeur annonce des informations IPv6.

```text
Router → RA → Client
```

### 🧠 À mémoriser

```text
RS = demande
RA = annonce
```

---

# 🌫️ 8.4 Stateless DHCPv6

Dans le mode Stateless :

```text
SLAAC → adresse IPv6
DHCPv6 → informations supplémentaires
```

Par exemple, DHCPv6 peut fournir des informations comme le DNS.

---

# 🗄️ 8.5 Stateful DHCPv6

Dans le mode Stateful :

```text
DHCPv6 → adresse IPv6 + informations de configuration
```

Le serveur DHCPv6 maintient l'état des adresses attribuées.

---

# 👋 8.6 NS et NA

Ces messages sont utilisés par **Neighbor Discovery**.

### 🔔 NS

**Neighbor Solicitation**

Utilisé notamment pour découvrir/vérifier un voisin et certaines fonctions de résolution d'adresse.

### 💬 NA

**Neighbor Advertisement**

Réponse à une sollicitation ou annonce concernant un voisin.

### 📌 À retenir

```text
RS → Router Solicitation
RA → Router Advertisement

NS → Neighbor Solicitation
NA → Neighbor Advertisement
```

---

# 🚩 8.7 Les flags IPv6

Dans les exercices Cisco, les flags du RA permettent notamment d'indiquer au client comment obtenir sa configuration.

À retenir conceptuellement :

```text
SLAAC → adresse construite automatiquement
DHCPv6 → informations fournies par serveur DHCPv6
```

---

# 9️⃣ Module 9 — FHRP

# ⚠️ 9.1 Problème de la passerelle unique

Imagine :

```text
PC
 |
Router 1
 |
Internet
```

Si Router 1 tombe :

```text
PC
 |
❌ Router 1
```

Le PC ne peut plus utiliser sa passerelle.

---

# 💡 9.2 Solution : FHRP

**FHRP = First Hop Redundancy Protocol**

Il permet à plusieurs routeurs de fournir une passerelle virtuelle redondante.

```text
              Virtual IP
                  |
          ┌───────┴───────┐
          |               |
      Router 1         Router 2
       Active            Standby
```

Le PC utilise la **Virtual IP** comme passerelle.

---

# 🔥 9.3 HSRP

**HSRP = Hot Standby Router Protocol**

Protocole Cisco.

Les rôles principaux sont :

```text
Active
Standby
```

Le routeur Active traite le trafic.

Le Standby peut prendre le relais si nécessaire.

---

# 🌐 9.4 VRRP

**VRRP = Virtual Router Redundancy Protocol**

C'est un protocole standardisé de redondance de passerelle.

Rôles courants :

```text
Master
Backup
```

---

# ⚖️ 9.5 GLBP

**GLBP = Gateway Load Balancing Protocol**

Protocole Cisco.

En plus de la redondance, GLBP peut permettre une forme de **répartition de charge** entre plusieurs routeurs.

---

# 🔥 Comparaison FHRP

```text
HSRP → Cisco → Active / Standby
VRRP → Standard → Master / Backup
GLBP → Cisco → redondance + load balancing
```

---

# 🧠 SECTION EXAMEN — À SAVOIR PAR CŒUR

## 🚦 1. Access vs Trunk

```text
ACCESS
→ généralement un seul VLAN
→ PC vers Switch

TRUNK
→ plusieurs VLAN
→ Switch vers Switch
→ Switch vers Routeur
```

---

## 🔀 2. VLAN vs Routage Inter-VLAN

```text
VLAN
→ sépare les réseaux au niveau Layer 2

Inter-VLAN Routing
→ permet aux VLAN différents de communiquer
→ nécessite Layer 3
```

---

## 🌳 3. STP

```text
STP
→ empêche les boucles Layer 2
→ élit un Root Bridge
→ peut bloquer un chemin redondant
```

---

## 🔗 4. EtherChannel

```text
Plusieurs câbles physiques
        ↓
   Port-Channel
        ↓
Un lien logique
```

LACP :

```text
active + active → OK
active + passive → OK
passive + passive → ❌
```

PAgP :

```text
desirable + desirable → OK
desirable + auto → OK
auto + auto → ❌
```

---

## 📡 5. DHCP

```text
D → Discover
O → Offer
R → Request
A → ACK
```

---

## 🌍 6. IPv6

```text
RS → Router Solicitation
RA → Router Advertisement
NS → Neighbor Solicitation
NA → Neighbor Advertisement
```

---

## 🛡️ 7. FHRP

```text
HSRP → Cisco
VRRP → Standard
GLBP → Cisco + Load Balancing
```

---

# 💻 COMMANDES CISCO À MÉMORISER

## 🧰 Général

```bash
enable
configure terminal
hostname SW1
no ip domain-lookup
copy running-config startup-config
```

## 🔍 Vérification

```bash
show running-config
show startup-config
show version
show ip interface brief
```

## 🏷️ VLAN

```bash
vlan 10
name ADMIN
interface fa0/1
switchport mode access
switchport access vlan 10
show vlan brief
```

## 🚦 Trunk

```bash
interface g0/1
switchport mode trunk
show interfaces trunk
```

## 🔁 Inter-VLAN

```bash
interface g0/0.10
encapsulation dot1Q 10
ip address 192.168.10.1 255.255.255.0
```

## 🌳 STP

```bash
show spanning-tree
spanning-tree vlan 10 priority 4096
```

## 🔗 EtherChannel

```bash
interface range g0/1-2
channel-group 1 mode active
show etherchannel summary
```

## 📡 DHCP

```bash
ip dhcp excluded-address 192.168.1.1 192.168.1.10
ip dhcp pool LAN
network 192.168.1.0 255.255.255.0
default-router 192.168.1.1
dns-server 8.8.8.8
show ip dhcp binding
show ip dhcp pool
```

## 🔁 DHCP Relay

```bash
interface g0/0
ip helper-address 192.168.100.10
```

---

# ⚠️ ERREURS FRÉQUENTES AU CC

### ❌ 1. Oublier `no shutdown`

Une interface peut rester administrativement désactivée.

```bash
no shutdown
```

### ❌ 2. Confondre Access et Trunk

```text
PC → Access
Switch ↔ Switch → Trunk
```

### ❌ 3. Oublier la passerelle

Un PC qui doit communiquer avec un autre réseau doit avoir une passerelle correcte.

### ❌ 4. Mauvais VLAN sur un port

Vérifier :

```bash
show vlan brief
```

### ❌ 5. Trunk incorrect

Vérifier :

```bash
show interfaces trunk
```

### ❌ 6. EtherChannel incompatible

Vérifier :

```bash
show etherchannel summary
```

### ❌ 7. STP

Ne pas confondre :

```text
Root Bridge
Root Port
Designated Port
```

### ❌ 8. DHCP

Mémoriser absolument :

```text
DORA
Discover → Offer → Request → ACK
```

---

# 🎯 MÉTHODE POUR RÉSOUDRE UN EXERCICE PACKET TRACER

Quand tu reçois une topologie, travaille dans cet ordre :

## 1️⃣ Étape 1 — Vérifier les interfaces

```bash
show ip interface brief
```

## 2️⃣ Étape 2 — Configurer les VLAN

```bash
show vlan brief
```

## 3️⃣ Étape 3 — Configurer les ports Access

```bash
switchport mode access
switchport access vlan X
```

## 4️⃣ Étape 4 — Configurer les Trunks

```bash
switchport mode trunk
```

## 5️⃣ Étape 5 — Configurer Inter-VLAN

Routeur :

```bash
encapsulation dot1Q X
ip address ...
```

ou switch Layer 3 :

```bash
interface vlan X
ip address ...
ip routing
```

## 6️⃣ Étape 6 — Vérifier STP

```bash
show spanning-tree
```

## 7️⃣ Étape 7 — Vérifier EtherChannel

```bash
show etherchannel summary
```

## 8️⃣ Étape 8 — Vérifier DHCP

```bash
show ip dhcp binding
```

## 9️⃣ Étape 9 — Tester

```bash
ping
```

Puis vérifier le chemin si nécessaire :

```bash
traceroute
```

---

# 📝 QUESTIONS TRÈS IMPORTANTES À SAVOIR EXPLIQUER

### ❓ Q1. Pourquoi utiliser un VLAN ?

Pour séparer logiquement un réseau, réduire les domaines de broadcast et améliorer l'organisation et la segmentation.

### ❓ Q2. Quelle est la différence entre Access et Trunk ?

Un port Access transporte généralement un seul VLAN, tandis qu'un Trunk transporte plusieurs VLAN.

### ❓ Q3. Pourquoi utiliser STP ?

Pour éviter les boucles de niveau 2 dans un réseau avec des chemins redondants.

### ❓ Q4. Qu'est-ce que le Root Bridge ?

C'est le switch élu comme référence par STP pour le calcul des chemins.

### ❓ Q5. Pourquoi utiliser EtherChannel ?

Pour regrouper plusieurs liens physiques en un lien logique afin d'augmenter la capacité et d'apporter de la redondance.

### ❓ Q6. Qu'est-ce que DORA ?

```text
Discover
Offer
Request
Acknowledgment
```

C'est le processus classique d'obtention d'une configuration IPv4 par DHCP.

### ❓ Q7. Qu'est-ce que SLAAC ?

Un mécanisme permettant à un hôte IPv6 de configurer automatiquement son adresse à partir des informations annoncées par le routeur.

### ❓ Q8. Pourquoi utiliser FHRP ?

Pour fournir une passerelle par défaut redondante en cas de panne d'un routeur.

---

# ⭐ FICHE ULTRA-RAPIDE AVANT L'EXAMEN

```text
VLAN
↓
Segmentation Layer 2

ACCESS
↓
1 VLAN

TRUNK
↓
Plusieurs VLAN

INTER-VLAN
↓
Communication entre VLAN
↓
Layer 3

STP
↓
Empêche les boucles

ROOT BRIDGE
↓
Référence STP

ETHERCHANNEL
↓
Plusieurs liens → 1 lien logique

LACP
↓
Active / Passive

PAgP
↓
Desirable / Auto

DHCPv4
↓
DORA

IPv6
↓
RS / RA / NS / NA

SLAAC
↓
Auto-configuration IPv6

FHRP
↓
Redondance de passerelle

HSRP
↓
Cisco

VRRP
↓
Standard

GLBP
↓
Cisco + Load Balancing
```

# 🏆 Les 10 éléments à connaître absolument

1. 🎛️ **Modes Cisco IOS**
2. 📚 **Table MAC : Learning / Forwarding / Flooding**
3. 🏷️ **VLAN et domaine de broadcast**
4. 🚦 **Access vs Trunk**
5. 🛣️ **Router-on-a-Stick**
6. 🌳 **STP + Root Bridge + rôles des ports**
7. 🔗 **EtherChannel + LACP/PAgP**
8. 📡 **DHCPv4 + DORA**
9. 🌍 **SLAAC + DHCPv6 + RS/RA**
10. 🛡️ **FHRP : HSRP / VRRP / GLBP**