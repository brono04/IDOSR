Un **agent de relais  relie les clients et serveurs de **sous-réseaux différents .
- 🖥️ **PC → Switch** → **câble droit (straight-through)**
- 🖥️ **PC → Router** → **câble droit**
- 🔀 **Switch → Router** → **câble droit**
- 🔀 **Switch → Switch** → traditionally **câble croisé (crossover)**
- 🖥️ **PC → PC** → traditionally **câble croisé**
**Different devices** → droit**  
**Same devices → croisé

CDP : Protocole propriétaire Cisco pour découvrir les voisins directement connectés.

for desactive tous port
```cs
Switch(config)# interface range fastethernet 0/1 - 4
Switch(config)# sh
```
# 📘 CHAPITRE 1 — Configurer les périphériques réseaux


## 🖥️ Configuration d'un commutateur

### 🔄 La séquence de démarrage du commutateur

- **POST** → Test du matériel.
- **Bootloader** → Démarre depuis la ROM.
- **CPU** → Initialise le processeur.
- **Flash** → Initialise la mémoire Flash.
- **IOS** → Charge l’IOS dans la RAM.

> **bootloader** : un petit programme stocké dans la ROM

**La commande boot system**

```css
Router(config)# boot system flash:/nom_de_l_image.bin
```

---

### 💡 LED du commutateur

- 🟢 Vert : fonctionnement normal (la signification exacte dépend du modèle du switch)
- 🟠 Orange : problème/état particulier (la signification exacte dépend du modèle du switch)

| LED       | Rôle                                                |
| --------- | --------------------------------------------------- |
| **SYST**  | Indiquer l’état général du système.                 |
| **RPS**   | Indiquer l’état de l’alimentation redondante.       |
| **STAT**  | Indiquer l’état et l’activité du port.              |
| **DUPLX** | Indiquer le mode duplex : Half ou Full-duplex.      |
| **SPEED** | Indiquer la vitesse de connexion du port.           |
| **PoE**   | Indiquer l’alimentation d’un appareil via Ethernet. |

| LED                               | Signification                                                                                         |     |
| --------------------------------- | ----------------------------------------------------------------------------------------------------- | --- |
| **SYST (Système)**                | 🟢 Vert = OK \| 🟠 Orange = Défaut ou erreur d'alimentation                                           |     |
| **RPS (Alimentation redondante)** | Indique l'état de l'alimentation de secours → 🟢 Vert clignotant : RPS est activé mais pas disponible |     |
| **STAT (État)**                   | 🟢 Vert clignotant = Trafic de données actif                                                          |     |
| **DUPLX (Duplex)**                | ⚫ Éteint = Semi-duplex (Half-Duplex) \| 🟢 Vert = Full-duplex                                         |     |
| **SPEED (Vitesse)**               | Indique la vitesse du port (**10 / 100 / 1000 Mbit/s**) via des séquences de clignotement             |     |
| **PoE (Power over Ethernet)**     | Indique si le port alimente un appareil (téléphone IP, caméra)                                        |     |

> **Le PoE** : envoie les données et l'énergie électrique via un seul câble Ethernet.

---

### 🛟 Récupération après une panne de système

1. Accéder au mode Bootloader (ROMMON)
2. Initialiser le système de fichiers

```css
switch: flash_init   # Active la mémoire flash
switch: dir flash:    # Liste les fichiers stockés en mémoire pour trouver IOS
```

3. Récupérer l'IOS (2 options)

```css
switch: boot flash:/nom_de_l_image.bin
```

---

### 🔐 Accès à la gestion du commutateur

- **Configuration initiale :** Utilisation obligatoire d'un **câble console** connecté à un PC.
- **Gestion à distance (Même réseau) :** Configuration d'une adresse IP et d'un masque sur l'**interface virtuelle (SVI)**.
- **Gestion à distance (Réseau distant) :** Configuration obligatoire d'une **passerelle par défaut** (default gateway).

---

### 🔄 La communication en mode duplex

#### Mode Duplex

- **Full-Duplex :** Transmission et réception **simultanées**. Cela augmente la bande passante réelle.
- **Half-Duplex :** Transmission dans **un seul sens à la fois**. Cela réduit les performances et provoque des collisions.
- **Risque :** Un mauvais réglage du duplex ou de la vitesse entraîne des **problèmes de connectivité**.

#### 🔌 Fonction Auto-MDIX

- **Rôle :** Détecte automatiquement le type de câble (**droit ou croisé**) et adapte la connexion.
- **Condition :** La vitesse et le duplex de l'interface sur **auto**.
- **Vérification :**

```css
Switch# show controllers ethernet-controller fastEthernet 0/1 phy | include Auto-MDIX
# ---> permet de vérifier si la fonction Auto-MDIX est activée
```

---

### 📋 Commandes de vérification du commutateur

|Commande|Description|
|:--|:--|
|`show running-config`|Affiche la configuration en cours d'exécution (dans la RAM).|
|`show startup-config`|Affiche la configuration de démarrage sauvegardée (dans la NVRAM).|
|`show version`|Affiche l'état matériel, la version du système d'exploitation (IOS) et le temps d'activité.|
|`show flash`|Liste le contenu de la mémoire Flash (fichiers `.bin`, configurations, etc.).|
|`show history`|Affiche l'historique des dernières commandes exécutées dans le terminal.|
|`show interfaces [interface-id]`|Affiche le statut global et les statistiques de trafic détaillées d'une interface.|
|`show ip interface [interface-id]`|Affiche les paramètres réseau et l'état IPv4 d'une interface spécifique.|
|`show ipv6 interface [interface-id]`|Affiche les paramètres réseau et l'état IPv6 d'une interface spécifique.|
|`show mac-address-table`|Affiche la table des adresses MAC apprises et leurs ports correspondants.|

---

### 🔒 Accès à distance sécurisé

**1️⃣ Affecter une IP à un VLAN**

```css
Switch(config)# interface vlan 1
Switch(config-if)# ip address 192.168.1.1 255.255.255.0
Switch(config-if)# no shutdown
```

**2️⃣ Affecter une IP à une interface**

```css
Routeur(config)# interface fastEthernet 0/0
Routeur(config-if)# ip address 192.168.1.2 255.255.255.0
```

**3️⃣ Configurer Telnet (Accès à distance non sécurisé)**

```css
Switch(config)# line vty 0 4
Switch(config-line)# password Carl
Switch(config-line)# login
```

**4️⃣ Configurer SSH (Accès à distance sécurisé)**

```css
Switch(config)# username Glenn password [votre_mot_de_passe]
Switch(config)# ip domain-name Formation
Switch(config)# crypto key generate rsa
Switch(config)# ip ssh version 2
Switch(config)# line vty 0 4
Switch(config-line)# login local
Switch(config-line)# transport input ssh

7 line
```

---

## 🌐 Configuration de base d'un routeur

_in pdf named cisco_tp_

**Double pile :** Coexistence **simultanée d'IPv4 et IPv6** sur les mêmes interfaces du routeur.

- Grandit utilisation IPv6 sans arrêt IPv4.

### ⚙️ Configuration d'une interface physique (LAN/WAN)

Trois commandes essentielles sont nécessaires :

- **Adresse IP :** `ip address [adresse-ipv4] [masque]` et/ou `ipv6 address [adresse-ipv6]/[préfixe]`
- **Activation :** `no shutdown` (sort l'interface de l'état désactivé)
- **Description :** `description [texte]` (aide à l'identification, maximum 240 caractères)

### 🔄 Interface de Bouclage (Loopback)

Interface logique et virtuelle interne au routeur toujours active.

- **Rôle :** Elle offre une adresse IP stable pour administrer le routeur (SSH/Telnet) ou pour des protocoles comme **OSPF** ou **BGP**.

|Tâche|Commande IPv4|Commande IPv6|
|---|---|---|
|Résumé des interfaces _(Adresse + État global)_|`show ip interface brief`|`show ipv6 interface brief`|
|Détail complet d'une interface _(Adresses + Paramètres)_|`show ip interface [id]`|`show ipv6 interface [id]`|
|Configuration en cours _(Commandes appliquées au port)_|`show running-config interface [id]`|_(Identique pour IPv4/IPv6)_|
|Table de routage _(Réseaux stockés en RAM)_|`show ip route`|`show ipv6 route`|
### 🧰 Filtres

- **`section |`** → عرض قسم كامل.
- **`include |`** → عرض الأسطر التي تحتوي على كلمة.
- **`exclude |`** → إخفاء الأسطر التي تحتوي على كلمة.
- **`begin |`** → البدء من سطر يحتوي على كلمة.

---

# 📘 CHAPITRE 2 — PARTIE 1 : Appliquer les concepts de commutation

> ⏱️ Durée : 1h | 🎯 Objectif : Comprendre les concepts de commutation

---

## 🗂️ Plan du chapitre

1. Concepts de commutation

---

## 1️⃣ Concepts de commutation

### 📦 Transfert de trame

- **Ingress** : entrée dans l'interface
- **Egress** : sortie de l'interface
- Le commutateur transfère selon l'**interface d'entrée** et l'**adresse MAC de destination**.
- Il utilise sa **table d'adresses MAC** pour décider.

⚠️ Un commutateur ne renvoie **jamais** le trafic sur l'interface où il l'a reçu.

### 🗃️ Table d'adresses MAC (table CAM)

Le commutateur construit cette table (Content Addressable Memory) en enregistrant l'adresse MAC source avec le port de réception.

### 🔄 Méthode d'apprentissage et de transmission (2 étapes)

| Étape             | Action                                         |
| ----------------- | ---------------------------------------------- |
| **1. Apprendre**  | Lire le MAC source → l’ajouter à la table.     |
| **2. Transférer** | Lire le MAC destination → envoyer au bon port. |

Les commutateurs utilisent des **ASIC** (circuits intégrés spécifiques) pour des décisions rapides, selon deux méthodes :

- **Commutation par stockage et transfert (Store-and-Forward)**
- **Commutation par coupure (Cut-Through)**

### ⚡ Comparaison des méthodes de commutation

| **Store-and-Forward**               | **Cut-Through**                              |
| ----------------------------------- | -------------------------------------------- |
| Reçoit toute la trame.              | Transfère la trame rapidement.               |
| Vérifie les erreurs.                | Ne vérifie pas toute la trame.               |
| Stocke la trame avant de l’envoyer. | Envoie dès qu’il connaît le MAC destination. |
### 💥 Domaines de collision
==> une zone où des collisions sont possibles
- **Duplex intégral (Full duplex)** → pas de collisions.
- **Semi-duplex** → collisions possibles.
- **Auto-négociation** → configure automatiquement le mode et la vitesse.
### 📢 Domaines de diffusion
==> une zone où un message est envoyé à tous les appareils.
- Un **broadcast** atteint les appareils du même réseau local.
- Un **routeur (couche 3)** sépare les domaines de diffusion.
- Trop de broadcasts → **congestion**.
### 🚦 Réduction de la congestion

|Caractéristique|Fonction|
|---|---|
|**Port rapide**|Transfert rapide des données.|
|**Commutation rapide**|Traite les trames rapidement.|
|**Grand tampon**|Stocke temporairement les trames.|
|**Plus de ports**|Connecte plus d’appareils.|
# 📘 CHAPITRE 3 — PARTIE 1 : Mettre en œuvre des VLAN

> ⏱️ Durée : 2h | 🎯 Objectif : Configurer les VLAN et le routage inter-VLAN

---

## 🗂️ Plan du chapitre

1. Aperçu des VLAN
2. Configuration des VLAN
3. Routage inter-VLAN
4. Protocole DTP

---

## 1️⃣ Aperçu des réseaux locaux virtuels (VLAN)

### 📌 Définition

Un *VLAN* (Virtual Local Area Network) est une technique de réseau qui divise un réseau en plusieurs réseaux virtuels indépendants (مستقلين).

### ✅ Avantages

- **Sécurité** 🛡️ : ils ne peuvent pas communiquer avec d’autres VLANs **sans passer par un routeur**
- **Performance** ⚡ : Réduit les diffusions (Broadcasts) et augmente la vitesse du réseau.
- **Flexibilité** 🔧 : Permet de déplacer des appareils entre VLANs sans changer les câbles.
- **Réduction du domaine de diffusion** : Chaque VLAN constitue un domaine de broadcast distinct.

### 🏷️ Types de VLAN

| **Type de VLAN**    | **Rôle / Utilisation**                                                                                 | **Exemple**                       |
| ------------------- | ------------------------------------------------------------------------------------------------------ | --------------------------------- |
| **VLAN de donnée**  | Transporte le trafic normal des utilisateurs                                                           | Département RH, IT                |
| **VLAN par défaut** | VLAN initial de tous les ports sur un switch                                                           | VLAN 1                            |
| **VLAN **Natif      | Il transporte le trafic non tagué sur un  lien Trunk.                                                  | Tous les VLAN opérationnels       |
| **VLAN de gestion** | Utilisé pour gérer le switch à distance (Telnet, SSH…)                                                 | VLAN pour l’administration réseau |
| **VLAN voix**       | un réseau local virtuel spécialisé au transport du trafic vocal  tel que les données VoIP (voix sur IP | =======                           |

### 🔗 Trunk (liaison entre commutateurs)

- Un **trunk** est une liaison point-à-point entre deux périphériques réseau (switch,routeur,Firewalls).
- Un trunk peut transporter plusieurs VLANs avec **802.1Q** ; les VLANs autorisés dépendent de la configuration.
###  Réseau sans/avec VLAN

**Sans VLAN**, tout le trafic (monodiffusion, multidiffusion, diffusion) est envoyé à **tous les périphériques** du réseau.
**Avec VLAN**, ce trafic est **bloqué au sein d'un seul VLAN**.
### 🏷️ Étiquetage 802.1Q (VLAN Tagging)

- En-tête 802.1Q = **4 octets**
- 802.1Q: Protocole qui ajoute un tag VLAN aux trames sur un trunk.
- VLAN native: VLAN dont les trames sont envoyées sans tag sur un trunk.
#### **L'essentiel du Trunk 802.1Q et VLAN Natif**

- **Trunk 802.1Q** : Taggue tous les VLANs, **sauf le VLAN natif**.
- **VLAN 1** : souvent le VLAN natif par défaut sur les équipements Cisco, sauf modification de configuration.
- **Règle** : Même VLAN natif obligatoire des deux côtés d'un trunk.
- **Indépendance** : Chaque liaison trunk se configure séparément.

---

## 2️⃣ Configuration des VLAN

### 🛠️ Création d'un VLAN

```
Switch# configure terminal
Switch(config)# vlan <vlan-id>
Switch(config-vlan)# name <vlan-name>
Switch(config-vlan)# end
```

📁 Les détails du VLAN sont stockés dans le fichier `vlan.dat`

### 🔌 Attribution d'un port à un VLAN

```
Switch# configure terminal
Switch(config)# interface <interface-id>
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan <vlan-id>
Switch(config-if)# end
```

### 🔍 Vérification

```
Switch# show vlan [brief | id vlan-id | name vlan-name | summary]
```

### ✏️ Modification / Suppression

- Modifier le port : `switchport access vlan <vlan-id>` (à nouveau)
- Revenir au VLAN 1 : `no switchport access vlan`
- Supprimer un VLAN : `no vlan <vlan-id>` ⚠️ **Réaffecter les ports avant suppression**
- Supprimer **tous** les VLAN : `delete flash:vlan.dat` puis **reload**

### 🔗 Configuration d'un trunk

```
Switch# configure terminal
Switch(config)# interface <interface-id>
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk native vlan <vlan-id>
Switch(config-if)# switchport trunk allowed vlan <vlan-list>
Switch(config-if)# end
Switch# show interface <interface-id> switchport
```

### 📡 Protocole VTP (VLAN Trunking Protocol)

un protocole de couche 2 propriétaire(ملكية خاصة) de Cisco utilisé pour gérer et synchroniser les configurations de VLAN

| Mode             | Fonction                                                                  |
| ---------------- | ------------------------------------------------------------------------- |
| **Server**       | Crée, modifie et diffuse les VLAN à tout le domaine.                      |
| **Client**       | Reçoit et applique la liste des VLAN                                      |
| **Transparent**  | VLAN locaux , mais transmet les mises à jour aux autres.                  |
| **VTP password** | Sécurise le domaine en empêchant(منع) l'ajout de faux serveurs.           |
| **VTP pruning**  | Bloque le trafic inutile sur les trunks pour économiser la bande passante |

```
# Serveur
Switch1(config)# vtp domain monDomain
Switch1(config)# vtp password monMotDePasse
Switch1(config)# vtp mode server
Switch1(config)# vtp pruning

# Client (identique, sauf vtp client)
Switch2(config)# vtp domain monDomain
Switch2(config)# vtp password monMotDePasse
Switch2(config)# vtp mode client

#mode transparent
Switch3(config)# vtp mode transparent
```

---
## 3️⃣ Routage inter-VLAN

### ❓ Pourquoi ?

Les VLAN différents ne communiquent pas directement. Il faut un **routeur** ou un **switch couche 3**.
### 🛣️ 3 méthodes

|Méthode|Description|
|---|---|
|**Legacy**|Une interface physique par VLAN.|
|**Router-on-a-Stick**|Une interface trunk + sous-interfaces.|
|**Switch couche 3 (SVI)**|Une SVI pour chaque VLAN.|
### ⚙️ Router-on-a-Stick

- Interface du routeur en **trunk 802.1Q**.
- Une **sous-interface + IP** par VLAN.
- Le routeur fait le **routage entre VLAN**.
### 🏆 Switch couche 3 + SVI

- **Plus rapide**.
- **Latence plus faible**.
- Pas besoin d'un routeur externe.
- **Plus cher**.
### 🩺 Dépannage

|Problème|Vérification|
|---|---|
|VLAN|`show vlan brief`|
|Trunk|`show interfaces trunk`|
|Port access|`show interfaces switchport`|
|Routeur|`show ip interface brief`|

---
## 4️⃣ DTP

**DTP (Dynamic Trunking Protocol)** : protocole Cisco qui **négocie automatiquement le trunk** entre switches.
### 📌 Modes

| Mode                | Rôle                        |
| ------------------- | --------------------------- |
| `access`            | Port d'accès                |
| `dynamic auto`      | Attend une demande de trunk |
| `dynamic desirable` | Cherche à créer un trunk    |
| `trunk`             | Trunk permanent             |

| **Mode 1**        | **Mode 2**        | **Résultat**   |
| ----------------- | ----------------- | -------------- |
| Access            | Access            | ❌ Pas de trunk |
| Access            | Dynamic Auto      | ❌ Pas de trunk |
| Access            | Dynamic Desirable | ❌ Pas de trunk |
| Trunk             | Trunk             | ✅ Trunk        |
| Trunk             | Dynamic Auto      | ✅ Trunk        |
| Trunk             | Dynamic Desirable | ✅ Trunk        |
| Dynamic Auto      | Dynamic Auto      | ❌ Pas de trunk |
| Dynamic Auto      | Dynamic Desirable | ✅ Trunk        |
| Dynamic Desirable | Dynamic Desirable | ✅ Trunk        |

**Désactiver DTP :**

```
switchport nonegotiate
```
**Vérifier :**

```
show dtp interface
```

💡 **À retenir :** `access` = VLAN unique → `trunk` = plusieurs VLAN.


# 📘 CHAPITRE 1 — PARTIE 2 : Étudier l'évolutivité du réseau

> ⏱️ Durée : 1.5h | 🎯 Objectif : Concevoir un réseau évolutif avec le bon choix des équipements

---

## 🗂️ Plan du chapitre

1. Conception de réseau évolutif
2. Sélection des périphériques réseau

---
## 1️⃣ Conception d'un réseau évolutif

### 📌 Évolutivité

Capacité d'un réseau à **grandir sans problème**.

Elle repose sur :

- 🔁 **Redondance** → éviter les pannes.
- 🔗 **Liens multiples** → augmenter la capacité.
- 🛣️ **Routage évolutif** → ex. OSPF.
- 📶 **Wi-Fi** → étendre facilement le réseau.

### 🏗️ Conception hiérarchique

**Cœur → Distribution → Accès**

- **Accès** → connecte les utilisateurs et appareils.
- **Distribution** → routage inter-VLAN + contrôle du trafic.
- **Cœur** → transporte les données rapidement.
==> can be `couch d'acces` and `couch coeur de reseaux` for it use Switches Layer 3 

---

## 2️⃣ Sélection des périphériques réseau

### 📐 Types de commutateurs

|Type|Description|
|---|---|
|**Fixe**|Configuration non modifiable.|
|**Modulaire**|Cartes remplaçables.|
|**Empilable**|Plusieurs switches fonctionnent comme un seul.|

### 🔢 Densité de ports

Nombre de ports disponibles : **12 / 24 / 48**, etc.

### ⚡ Débit

Vitesse de transmission des données : **Mbps / Gbps**.

### 🔌 PoE

Fournit l'**électricité + données** via le câble réseau.

### 🧠 Layer 3 Switching

Permet de faire du **routage IP rapide** avec des **ASIC**.

### 💼 Choisir un switch

- 💰 Coût
- 🔢 Nombre de ports
- ⚡ Vitesse
- 🔌 PoE
- 🛡️ Fiabilité

### 🚀 Choisir un routeur

- 💰 Coût
- 🔌 Types d'interfaces
- ⚡ Performance / débit
- 🛡️ Sécurité

# 📘 CHAPITRE 2 — PARTIE 2 : Implémenter la redondance dans les réseaux commutés sans boucle

> ⏱️ Durée : 2.5h | 🎯 Objectif : Comprendre le fonctionnement du protocole STP et configurer PVST+

---

## 🗂️ Plan du chapitre

1. Concepts du protocole Spanning Tree (STP)
2. Configuration du protocole STP

---
## 1️⃣ STP (Spanning Tree Protocol)

### 🔁 Pourquoi STP ?

- La **redondance** évite les pannes.
- Mais elle peut créer des **boucles de couche 2**.
- Les boucles peuvent provoquer une **tempête de diffusion** et surcharger le réseau.

### 📌 Rôle du STP

- 🚫 Évite les boucles.
- 🔒 Bloque les chemins redondants.
- 🔄 Réactive un chemin bloqué en cas de panne.

### 🧮  algorithm STA

1. Choisir le **Root Bridge**. : Le switch avec la **plus petite priorité**
2. Choisir les **Root Ports**. : Meilleur chemin vers le Root Bridge.
3. Choisir les **Designated Ports**. : le meilleur chemin vers le Root Bridge sur un segment.
4. Bloquer les autres ports. : Ils sont **bloqués** pour éviter les boucles réseau.

### 🆔 BID

Le **BID** est composé de :

1. **Priorité** → plus petite = meilleure.
2. **ID VLAN**.
3. **MAC** → plus petite = meilleure.

👉 **Le plus petit BID gagne.**

### 🪜 Choix du Root Port

1. **Coût du chemin** le plus faible.
2. **BID** du switch voisin le plus faible.
3. **ID du port** le plus faible.
### 🚦 États STP

| Étape | State      | État              | Rôle                         |
| ----- | ---------- | ----------------- | ---------------------------- |
| 1     | Blocking   | **Blocage**       | Ne transmet pas les données. |
| 2     | Listening  | **Écoute**        | Prépare la topologie.        |
| 3     | Learning   | **Apprentissage** | Apprend les adresses MAC.    |
| 4     | Forwarding | **Acheminement**  | Transmet les données.        |
| 5     | Disabled   | **Désactivé**     | Port désactivé.              |
### ⚡ Versions

- **RSTP** → STP plus rapide.
- **PVST+** → un STP par VLAN.

## 2️⃣ Configuration STP

```
configure terminal
spanning-tree mode rapid-pvst
interface <interface-id>
spanning-tree link-type point-to-point
end
```

### 🔍 Vérification

```
show spanning-tree
show spanning-tree summary
```

💾 **Sauvegarder :**

```
copy running-config startup-config
```

**🧠 À retenir :**

> **STP = éviter les boucles de couche 2 en bloquant les chemins redondants.**

# 📘 CHAPITRE 3 — PARTIE 2 : Configurer l'agrégation des liaisons

> ⏱️ Durée : 1h | 🎯 Objectif : Configurer l'agrégation des liaisons avec EtherChannel

---

## 🗂️ Plan du chapitre

1. Fonctionnement d'EtherChannel
2. Configuration d'EtherChannel

---

## 1️⃣ Fonctionnement d'EtherChannel

### 📌 Qu'est-ce qu'EtherChannel ?

**EtherChannel** est une **technologie Cisco** qui permet de **regrouper plusieurs liens Ethernet physiques** 🔌🔌 en **un seul lien logique** 🧠.
### ✅ Avantages

- **Plus rapide**
- **Un seul lien logique**
- **Config simple**
-  **Redondant** (cable en cas de pane)

### ⚠️ Restrictions d'implémentation(قيود التنفيذ)

- **8 ports max** par canal
- **6 canaux max** (sur Catalyst 2960)
- **Config cohérente** des deux côtés
- **Modif globale** (affecte tous les ports du canal)
### 🤝 Protocoles de négociation automatique

| Protocole | Type                   | Description                                   |
| --------- | ---------------------- | --------------------------------------------- |
| **PAgP**  | Propriétaire **Cisco** | créer automatiquement des liens EtherChannel  |
| **LACP**  | Norme **IEEE 802.3ad** | comme PAgP, compatible **multi-fournisseurs** |

💡 Une liaison EtherChannel **statique** (sans PAgP ni LACP) est aussi possible.

### ⚙️ Modes PAgP

|Mode|Comportement|
|---|---|
|**On**|Force le canal sans PAgP (pas d'échange de paquets)|
|**Desirable**|Négociation **active** — envoie des paquets PAgP|
|**Auto**|Négociation **passive** — répond mais n'initie pas|

**Résultat de la négociation PAgP :**

|S1|S2|Canal établi ?|
|---|---|---|
|On|On|✅ Oui|
|On|Desirable/Auto|❌ Non|
|Desirable|Desirable|✅ Oui|
|Desirable|Auto|✅ Oui|
|Auto|Desirable|✅ Oui|
|Auto|Auto|❌ Non|

### ⚙️ Modes LACP

|Mode|Comportement|
|---|---|
|**On**|Force le canal sans LACP|
|**Active**|Négociation **active** — envoie des paquets LACP|
|**Passive**|Négociation **passive** — répond mais n'initie pas|

**Résultat de la négociation LACP :**

|S1|S2|Canal établi ?|
|---|---|---|
|On|On|✅ Oui|
|On|Active/Passive|❌ Non|
|Active|Active|✅ Oui|
|Active|Passive|✅ Oui|
|Passive|Active|✅ Oui|
|Passive|Passive|❌ Non|

---

## 2️⃣ Configuration d'EtherChannel

- **Compatibilité requise** (toutes les interfaces)
- **Débit et duplex identiques**
- **Même VLAN** (ou même trunk)
- **Mêmes VLANs autorisés** (sur le trunk)
### 🛠️ Commande de configuration

```
Switch(config-if)# channel-group <number> mode <mode>
```

### 🔍 Vérification et dépannage

| Commande                         | Description                                 |
| -------------------------------- | ------------------------------------------- |
| `show interfaces port-channel`   | État global de l'interface de canal         |
| `show etherchannel summary`      | Une ligne d'infos par canal                 |
| `show etherchannel port-channel` | Infos sur une interface de canal spécifique |
| `show interfaces etherchannel`   | Rôle des interfaces physiques membres       |
# 📘 CHAPITRE 4 — PARTIE 2 : Comprendre le concept du FHRP

> ⏱️ Durée : 1h | 🎯 Objectif : Comprendre le fonctionnement des protocoles FHRP et configurer HSRP

---

## 🗂️ Plan du chapitre

1. Protocoles de redondance au premier saut
2. Fonctionnement et configuration du protocole HSRP

---

## 1️⃣ FHRP

**FHRP** → regroupe des routeurs de secours pour éviter les coupures internet si le principal tombe en panne.

### ⚠️ Problème

- Un hôte → **une seule passerelle**.
- Si elle tombe → **connexion interrompue**.

### 🎯 Solution

- Plusieurs routeurs utilisent une **IP virtuelle**.
- Si le routeur principal tombe → un autre **prend le relais**.

### 🔄 Failover

1. Routeur principal tombe.
2. Routeur de secours prend le relais.
3. Le réseau continue à fonctionner.

### 📋 Protocoles

- **HSRP** → Cisco.
- **VRRP** → standard ouvert.
- **GLBP** → Cisco + équilibrage de charge.

---

## 2️⃣ HSRP

### 📖 Fonctionnement

- **Active** → route le trafic.
- **Standby** → attend et prend le relais.

### 🏆 Priorité

- Par défaut : **100**.
- Priorité la plus élevée → **Active**.
- Égalité → **IP la plus élevée**.

```
standby 1 priority 110
standby 1 preempt
```

### ⏱️ Temporisateurs

- **Hello : 3 s**
- **Hold : 10 s**

### 🚦 États HSRP

| État        | Signification             |
| ----------- | ------------------------- |
| **Initial** | Démarrage du routeur.     |
| **Learn**   | Cherche l'IP virtuelle.   |
| **Listen**  | Écoute le groupe HSRP.    |
| **Speak**   | Participe à l'élection.   |
| **Standby** | Prêt à prendre le relais. |
#### 1.HSRP (Hot Standby Router Protocol)
```shell
 interface GigabitEthernet0/1
 ip address 192.168.1.2 255.255.255.0
 standby 1 version 2                  ! Sets HSRP version to 2
 standby 1 ip 192.168.1.1             ! Sets the Virtual IP address
 standby 1 priority 110               ! Higher priority becomes Active (default is 100)
 standby 1 preempt                    ! Allows this router to take over if its priority is higher
```
#### 2. VRRP (Virtual Router Redundancy Protocol)
```shell
interface GigabitEthernet0/1
 ip address 192.168.1.2 255.255.255.0
 vrrp 1 ip 192.168.1.1                ! Sets the Virtual IP address
 vrrp 1 priority 110                  ! Higher priority becomes Master (default is 100)
```
#### 3. GLBP (Gateway Load Balancing Protocol)
```shell
interface GigabitEthernet0/1
 ip address 192.168.1.2 255.255.255.0
 glbp 1 ip 192.168.1.1                ! Sets the Virtual IP address
 glbp 1 priority 110                  ! Higher priority becomes Active Virtual Gateway (AVG)
 glbp 1 preempt                        ! Allows this router to take over the AVG role
```

==> **GLBP** : fait de l’équilibrage de charge
# 📘 CHAPITRE 1 — PARTIE 3 : Comprendre le fonctionnement de DHCPv4

> ⏱️ Durée : 1h | 🎯 Objectif : Comprendre le fonctionnement de DHCPv4

---

## 🗂️ Plan du chapitre

1. Fonctionnement de DHCPv4

---

## 1️⃣ Concepts DHCPv4

### 📌 Serveur et client DHCPv4

- **protocole DHCP** : est un protocole réseau qui attribue automatiquement une adresse IPV4 aux appareils, sans configuration manuelle.
- **client DHCP** : Appareil demandant une configuration  
- **serveur DHCP** : Gère et attribue les adresses IPV4 

### ⚙️ Fonctionnement (mode client/serveur)

- **L'appareil loue (استئجار  ) l'IP** et doit demander au serveur pour la garder.
- **Si le temps expire**, l'IP est libérée pour un autre appareil.

### 🔄 Étapes pour obtenir un bail (4 étapes) OR Function DORA

1. *DISCOVER* – Le client recherche un serveur DHCP  
2. *OFFER* – Le serveur propose une adresse IP  
3. *REQUEST* – Le client accepte l’offre  
4. *ACK* – Le serveur confirme et envoie les paramètres  

### 🔁 Renouvellement du bail (2 étapes, avant expiration) 
==> 🔁 تجديد عقد الإيجار (خطوتان، قبل انتهاء الصلاحية)

1. **DHCPREQUEST** — envoyé directement au serveur d'origine
2. **DHCPACK** — le serveur confirme le bail

💡 Les messages DHCPOFFER/DHCPACK peuvent être en **monodiffusion ou diffusion** (RFC 2131).

---

## 2️⃣ Configurer un serveur Cisco IOS DHCPv4

### 🛠️ Étapes de configuration

1. **Exclure des adresses** : `ip dhcp excluded-address <low> [high]`
2. **Créer un pool** : `ip dhcp pool <pool-name>`
3. **Configurer le pool** : `network <réseau> [masque | /préfixe]` + `default-router <adresse>`

### 📋 Commandes de configuration du pool

| Tâche                     | Commande IOS                                        |
| ------------------------- | --------------------------------------------------- |
| Exclusion d'adresses IPv4 | `ip dhcp excluded-address [low] [high]`             |
| Nom du pool DHCPv4        | `ip dhcp pool <pool-name>`                          |
| Plage d'adresses          | `network <network-number> [mask \| /prefix-length]` |
| Passerelle par défaut     | `default-router <address> [address2...address8]`    |
| Serveur DNS               | `dns-server <address> [address2...address8]`        |
| Nom de domaine            | `domain-name <domain>`                              |
| Durée du bail             | `lease {days [hours [minutes]] \| infinite}`        |
### 🔍 Vérification DHCPv4

|Commande|Description|
|---|---|
|`show running-config \| section dhcp`|Commandes DHCPv4 configurées|
|`show ip dhcp binding`|Liaisons IPv4 ↔ MAC attribuées|
|`show ip dhcp server statistics`|Compteurs des messages envoyés/reçus|
|`ipconfig /all`|Paramètres TCP/IP côté client|

### ⛔ Désactiver le service DHCPv4

- Désactiver : `no service dhcp`
- Réactiver : `service dhcp`
- ⚠️ Effacer les liaisons ou redémarrer le service peut causer des **doublons d'IP** temporaires.

---
### 3️⃣ Relais DHCPv4

- **Utile si :** le serveur DHCP est sur un autre réseau.
- **Rôle :** le routeur reçoit le broadcast DHCP et le transmet au serveur.
- **Commande :** `ip helper-address <adresse_serveur>`
- **Vérification :** `show ip interface`
### 4️⃣ Routeur Cisco comme client DHCPv4

- **Utile pour :** connecter le routeur à un FAI (ISP).
- **Commande :** `ip address dhcp`
- **Résultat :** le routeur reçoit automatiquement une adresse IP du serveur DHCP.

# 📘 Résumé : SLAAC & DHCPv6

> Chapitre 2 — Partie 3 | 🎯 Objectif : comprendre SLAAC et DHCPv6, et configurer le service DHCPv6

---

## 1️⃣ GUA & Message RA 🚩

- **GUA** = Global Unicast Address (adresse IPv6 globale) 🌍
- **Link-Local** = créée automatiquement (`fe80::`)
- **RA** (Router Advertisement) = le routeur indique à l'hôte comment configurer son IPv6 📢

| Drapeau | Signification                                               |
| ------- | ----------------------------------------------------------- |
| **A=1** | Utiliser **SLAAC** 🤖                                       |
| **O=1** | **DHCPv6 sans état** (infos supplémentaires : DNS, domaine) |
| **M=1** | **DHCPv6 avec état** (GUA + infos réseau) 🖥️               |

---

## 2️⃣ SLAAC 🤖

- L'hôte crée **sa GUA tout seul** à partir du **préfixe** annoncé par le routeur ✅
- **Passerelle par défaut** = adresse **Link-Local du routeur** 🚪

|Message|Sens|Adresse|
|---|---|---|
|**RS** (Router Solicitation)|Hôte → Routeur|`ff02::2`|
|**RA** (Router Advertisement)|Routeur → Hôte|`ff02::1`|

- **DAD** (Duplicate Address Detection) : vérifie que l'adresse n'est pas déjà utilisée 🔍
- **EUI-64** : transforme l'adresse MAC (48 bits) en identifiant d'interface (64 bits) 🔄

---

## 3️⃣ DHCPv6 🖥️

|Type|Qui donne la GUA ?|Qui donne DNS, domaine... ?|Drapeaux RA|
|---|---|---|---|
|**Sans état** (Stateless)|SLAAC|DHCPv6|A=1, O=1, M=0|
|**Avec état** (Stateful)|DHCPv6|DHCPv6|A=0, M=1|

### 🔄 Processus

```
RS → RA → SOLICIT → ADVERTISE → REQUEST → REPLY
```

### 🔌 Ports UDP

- **Client** : UDP **546**
- **Serveur** : UDP **547**

💡 **Astuce** : Client = 546 (le plus petit), Serveur = 547 (le plus grand).

---

## 4️⃣ Rôles d'un routeur Cisco 🎭

- 🖥️ **Serveur DHCPv6**
- 💻 **Client DHCPv6**
- 🔁 **Relais DHCPv6** : transmet les requêtes vers un serveur distant

---

## 🛠️ Commandes importantes

|Commande|Utilité|
|---|---|
|`ipv6 address autoconfig`|Client SLAAC|
|`ipv6 address dhcp`|Client DHCPv6 avec état|
|`ipv6 nd other-config-flag`|DHCPv6 sans état (O=1)|
|`ipv6 nd managed-config-flag`|DHCPv6 avec état (M=1)|
|`ipv6 dhcp relay destination <IP>`|Configurer le relais|
|`show ipv6 interface brief`|Vérifier les interfaces|
|`show ipv6 dhcp binding`|Voir les adresses attribuées|

---
## ✅ À retenir

- **A** = _Auto_ → SLAAC
- **O** = _Other_ → infos supplémentaires (DNS) via DHCPv6
- **M** = _Managed_ → tout vient du serveur DHCPv6


# 📘 CHAPITRE 1 — PARTIE 4 : Sécuriser la couche 2 du réseau LAN

> ⏱️ Durée : 2h | 🎯 Objectif : Comprendre les concepts de sécurité de la couche 2 et configurer la sécurité des ports

---

## 🗂️ Plan du chapitre

1. Concepts de sécurité LAN
2. Configuration de la sécurité du commutateur

---
## 🔐 Sécurité LAN

### 1️⃣ Attaques courantes

- **DDoS** → plusieurs machines attaquent un serveur.
    
- **Violation de données** → vol de données.
    
- **Malware** → infection d’un appareil.
### 2️⃣ Protection

- **Antivirus / Pare-feu / HIPS** → protection des terminaux.
    
- **NGFW** → pare-feu avancé + filtrage.
    
- **NAC** → contrôle l’accès au réseau.
    
- **VPN** → connexion sécurisée à distance.
### 3️⃣ AAA

- **Authentification** → Vérifier l’identité de l’utilisateur.
- **Autorisation** → Définir les droits d’accès.
- **Comptabilité** → Enregistrer les activités de l’utilisateur.
- **RADIUS / TACACS+** → Serveurs utilisés pour l’AAA.
### 4️⃣ 802.1X

Contrôle l’accès au port avant d’autoriser la connexion.

- **Client** → demande l’accès.
    
- **Switch** → authentificateur.
    
- **Serveur** → vérifie l’identité.

### 5️⃣ Attaques de couche 2

- **MAC** → MAC Flooding.
    
- **VLAN** → VLAN Hopping.
    
- **DHCP** → DHCP Spoofing.
    
- **ARP** → ARP Spoofing.
    
- **IP/MAC** → Spoofing.
    
- **STP** → attaque/manipulation STP
### 6️⃣ Protection

|Technique|Protection|
|---|---|
|**Port Security**|MAC inconnues|
|**DHCP Snooping**|DHCP Spoofing|
|**DAI**|ARP Spoofing|
|**IPSG**|IP/MAC Spoofing|
### 7️⃣ Ports inutilisés

- Désactiver : `shutdown`
    
- Réactiver : `no shutdown`
    
- Plusieurs ports : `interface range`
### 8️⃣ Port Security

**Rôle :** limiter les adresses MAC autorisées sur un port.

```bash
switchport port-security
switchport port-security maximum <value>
```

#### 📌 Apprentissage MAC

- **Manuel** → MAC configurée.
    
- **Dynamique** → apprise automatiquement.
    
- **Sticky** → apprise + ajoutée à la configuration.
    

#### 🚨 Violation

- **Shutdown** → coupe le port (défaut).
    
- **Restrict** → bloque + alerte.
    
- **Protect** → bloque sans alerte.

## 🔐 Cisco Switch Security

### 🛡️ 1. Port Security

#### ⚙️ Activer

```
Switch(config)# interface F0/1
Switch(config-if)# switchport mode access
Switch(config-if)# switchport port-security
```

#### 🔑 MAC statique

```
switchport port-security mac-address 0002.16e2.b3cd
```

#### 🧠 MAC Sticky

```
switchport port-security mac-address sticky
```

#### 🔢 Limiter les MAC

```
switchport port-security maximum 1
```

#### 🚨 Violation

```
switchport port-security violation protect
```

- 🟢 `protect` → bloque seulement
    
- 🟠 `restrict` → bloque + message
    
- 🔴 `shutdown` → désactive le port (défaut)

#### 🔄 Réactiver le port

```
interface F0/1
shutdown
no shutdown
```

#### 🔎 Vérifier

```
show port-security interface F0/11
```

---

### 📡 2. CDP

#### ❌ Désactiver CDP

```
Switch(config-if)# no cdp enable
```

---

### 🛡️ 3. DHCP Snooping

#### ✅ Activer

```
Switch(config)# ip dhcp snooping
```

#### 🔒 Port DHCP légitime = Trusted

```
Switch(config-if)# ip dhcp snooping trust
```

#### 🚫 Autres ports = Untrusted

```
Switch(config-if)# no ip dhcp snooping trust
```

💡 **À retenir :** `trust` = port autorisé à envoyer des réponses DHCP.

- 📡 **CDP** : permet à un équipement Cisco de découvrir les équipements Cisco voisins.
- 🚨 **Violation** : une MAC non autorisée se connecte sur un port sécurisé.
- 🛡️ **DHCP Snooping** : protège contre les faux serveurs DHCP.
- 🟢 **DHCP légitime = Trusted** : le port connecté au vrai serveur DHCP est considéré comme fiable.

# here
# 📘 CHAPITRE 2 — PARTIE 4 : Concevoir et sécuriser un réseau local sans fil

> ⏱️ Durée : 4h | 🎯 Objectif : Concevoir et sécuriser un WLAN, le configurer et le dépanner

---

## 🗂️ Plan du chapitre

1. Présentation de la technologie sans fil
2. Fonctionnement d'un réseau WLAN
3. Mécanismes de sécurité WLAN
4. Configuration de réseau WLAN

---
### Éléments d’un WLAN

- **Access Point (AP) :** connecte les clients Wi-Fi au réseau.
- **Wireless client :** ordinateur, téléphone ou autre appareil sans fil.
- **SSID :** nom du réseau Wi-Fi.
- **BSSID :** identifiant radio du point d’accès, généralement basé sur une adresse MAC.
- **WLC :** contrôleur centralisant la gestion de plusieurs points d’accès dans une architecture contrôlée.
## 1️⃣ Présentation de la technologie sans fil

### 📶 Le réseau local sans fil (WLAN)

- Liberté de mouvement totale (maison et bureau).
- S'adapte à tous les nouveaux besoins.
### 📋 Normes 802.11

| Norme        | Fréquence    | Débit                  |
| ------------ | ------------ | ---------------------- |
| **802.11**   | 2,4 GHz      | jusqu'à 2 Mb/s         |
| **802.11a**  | 5 GHz        | jusqu'à 54 Mb/s        |
| **802.11b**  | 2,4 GHz      | jusqu'à 11 Mb/s        |
| **802.11g**  | 2,4 GHz      | jusqu'à 54 Mb/s        |
| **802.11n**  | 2,4 et 5 GHz | 150–600 Mb/s           |
| **802.11ac** | 5 GHz        | 450 Mb/s–1,3 Gb/s      |
| **802.11ax** | 2,4 et 5 GHz | Haute efficacité (HEW) |
### 📡 Bandes de fréquence

- **2,4 GHz (UHF)** → 802.11b/g/n/ax
- **5 GHz (SHF)** → 802.11a/n/ac/ax

### 📶 Catégories de points d'accès (AP)

- **AP autonomes**
- **AP basés sur un contrôleur** (WLC)

### 📡 Types d'antennes

| Type                   | Description                                         |
| ---------------------- | --------------------------------------------------- |
| **Omnidirectionnelle** | Couverture 360° — idéale maison/bureau              |
| **Directionnelle**     | Signal concentré (Yagi, parabole)                   |
| **MIMO**               | Jusqu'à 8 antennes, bande passante augmentée(مُعزز) |

---

## 2️⃣ Fonctionnement d'un réseau WLAN

### 🏗️ Modes de topologie 802.11

- **Mode infrastructure**
- **Mode ad hoc** ==> PC1 <---------->PC2
- **Partage de connexion** PC <---------- partage la conexion <------------ internet


![Mon Schéma](mode.png)

### 🔗 BSS et ESS

| Concept                        | Description                                                                                    |
| ------------------------------ | ---------------------------------------------------------------------------------------------- |
| **BSS** (Basic Service Set)    | Un seul AP interconnectant ses clients ; pas de communication entre BSS différents             |
| **ESS** (Extended Service Set) | Union de plusieurs BSS reliés par un système câblé ; les clients peuvent communiquer entre eux |

### 📡 CSMA/CA

Les WLAN utilisent **CSMA/CA** (accès multiple avec évitement de collision) pour gérer l'envoi des données.

### 🔗 Association client/AP

3 étapes, avec accord requis sur :

- **SSID** (nom du réseau)
- **Mot de passe**
- **Mode réseau** (norme 802.11)
- **Mode de sécurité** (WEP/WPA/WPA2)
- **Paramètres des canaux**

### 🔍 Découverte passive et active

- **Mode passif** : le client écoute le routeur.
- **Mode actif** : le client cherche le routeur.
### 🌐 CAPWAP

- Protocole Permet à un contrôleur (**WLC**) et gérer plusieurs AP/WLAN.
- **Sécurité :** Utilise le protocole **DTLS**.
- **Ports :** **UDP 5246 et 5247**.
- **Réseau :** Fonctionne en **IPv4** et **IPv6**.
### 🧩 Architecture MAC divisée (Split MAC)

| Fonctions AP (Borne) | Fonctions WLC (Contrôleur) |
|---|---|
| Gère les balises Wi-Fi | Gère l'authentification |
| Gère les accusés de réception | Gère le roaming (déplacement) |
| Gère les priorités | Traduit les protocoles |
| Chiffre/Déchiffre les données | Connecte le Wi-Fi au câble |
### 🔐 Chiffrement DTLS

- Sécurise le canal **AP ↔ WLC**.
- Activé par défaut pour le trafic de **contrôle/gestion**.
- Le **chiffrement des données** est désactivé par défaut (licence DTLS requise).
### 🔀 FlexConnect

| Mode         | Description                           |
| ------------ | ------------------------------------- |
| **Connecté** | WLC accessible, il gère tout.         |
| **Autonome** | WLC coupé, l'AP gère tout localement. |
### 📻 Saturation des canaux — techniques d'atténuation

- **Problème :** Une route saturée ralentit le Wi-Fi.
**Solutions :**

- **DSSS** : **Agrandit** la route.
- **FHSS**: Jumps channels to bypass **blocks**.
- **OFDM** : Divise la route en plusieurs couloirs

### 🎚️ Sélection des canaux

- **2,4 GHz** : canaux sans chevauchement → **1, 6, 11**
- **5 GHz** : canaux sans chevauchement → **36, 48, 60**

### 📐 Planifier un déploiement WLAN   (WLAN خطط لنشر شبكة)

- **Plan d'accès :** Disposition, nombre d'appareils, débits, canaux uniques, puissance.
---

## 3️⃣ Mécanismes de sécurité WLAN

### ⚠️ Menaces principales

- **Attaques DoS** (Déni de service) : evoiye plusier faux trafic pour le faire planter le system
- **Interception de données**
- **Attaques MITM**

### 🕵️ Masquage SSID et filtrage MAC (protections basiques)

- **Masquage SSID** : Masque(cache) le nom du réseau. Il faut s'y connecter manuellement.
- **Filtrage MAC** : autorise/refuse l'accès selon l'adresse MAC.
### 🔑 Méthodes d'authentification d'origine (802.11)

| Type               | Description                                                                  |
| ------------------ | ---------------------------------------------------------------------------- |
| **Système ouvert** | Aucun mot de passe (ex : Wi-Fi public), sécurité à la charge du client (VPN) |
| **Clé partagée**   | WEP/WPA/WPA2/WPA3 — mot de passe pré-partagé requis                          |

### 🔒 Méthodes d'authentification par clé partagée

|Méthode|Description|
|---|---|
|**WEP**|Chiffrement RC4, clé statique — ⚠️ **obsolète, à ne jamais utiliser**|
|**WPA**|TKIP (clé changée à chaque paquet)|
|**WPA2**|Chiffrement **AES** — le plus robuste historiquement|
|**WPA3**|Nouvelle génération, interdit les protocoles hérités, exige les **PMF** (cadres de gestion protégés)|

### 🏠 Authentification utilisateur à domicile

|Mode|Description|
|---|---|
|**Personnel**|Clé pré-partagée (**PSK**) — domicile/petite entreprise|
|**Entreprise**|Authentification **802.1X/EAP** via serveur **RADIUS**|

### 🔐 Méthodes de cryptage (WPA/WPA2)

| Protocole | Utilisé par | Description                                                     |
| --------- | ----------- | --------------------------------------------------------------- |
| **TKIP**  | WPA         | Compatible matériel ancien(قديم), protège les données.          |
| **AES**   | WPA2        | Plus sécurisé (CCMP), vérifie que le message n'est pas modifié. |

### 🏢 Authentification en entreprise (RADIUS)

Infos requises :

- Adresse IP du serveur RADIUS
- Ports UDP **1812** (authentification) / **1813** (comptabilité) — ou 1645/1646
- Clé partagée (AP ↔ serveur RADIUS)

### 🆕 WPA3 — 4 fonctionnalités

| Fonctionnalité      | Description                                               |
| ------------------- | --------------------------------------------------------- |
| **WPA3-Personnel**  | Protection contre force brute via **SAE**                 |
| **WPA3-Entreprise** | 802.1X/EAP + suite crypto **192 bits**                    |
| **Réseaux ouverts** | Chiffrement opportuniste **OWE** (sans authentification)  |
| **IoT Onboarding**  | Protocole **DPP** pour intégrer rapidement les objets IoT |

---

## 4️⃣ Configuration de réseau WLAN

### 🛠️ Configuration de base

**Réseau (routeur) :**

1. Connexion via navigateur
2. Modifier le mot de passe admin
3. Reconnexion avec le nouveau mot de passe
4. Modifier la plage DHCP par défaut
5. Renouveler l'IP
6. Reconnexion avec la nouvelle IP

**Sans fil :**

1. Vérifier les paramètres par défaut
2. Choisir le mode réseau (norme 802.11)
3. Configurer le **SSID**
4. Choisir un canal **sans chevauchement**
5. Configurer le mode de sécurité (Open, WPA, WPA2 Personal/Enterprise)
6. Configurer la **phrase secrète**

### 🕸️ Réseau maillé sans fil (Mesh)

- Recommandé au-delà de 45m (intérieur) ou 90m (extérieur).
- Ajouter des appareils identiques sur des canaux différents pour éviter les bugs de signal.
### 🌍 NAT et redirection de port

- **NAT IPv4** : IP publique côté FAI, IP privée côté LAN.
- **Redirection de port** : ouvre des ports spécifiques pour certaines applications.
- **Déclenchement de port** : transfert temporaire de ports entrants.
- **QoS** : disponible sur de nombreux routeurs sans fil.
### 🖥️ Accès au WLC (contrôleur WLAN)

- Configuration via navigateur (HTTP/HTTPS) à l'IP du contrôleur, ou via **Telnet/SSH**.
- Nécessite un compte utilisateur de gestion.
### 🔌 Ports du WLC

|Port|Fonction|
|---|---|
|**Service Port**|Gestion système, un seul VLAN, connecté en mode **Access**|
|**Distribution System Port**|Trafic des AP + gestion, connecté en mode **Trunk**, supporte **LAG**|
|**Console Port**|9600 bauds, 8 bits données, 1 bit d'arrêt|
|**Redundancy Port**|Connecte un autre contrôleur pour la **haute disponibilité**|
### 🧩 Interfaces logiques du WLC

|Interface|Fonction|
|---|---|
|**Gestion**|Auth RADIUS, communication inter-contrôleurs, Web/SSH, SNMP, NTP, syslog|
|**Redondance**|Utilisée par le contrôleur de secours|
|**Virtuelle**|Relais DHCP, authentification Web, terminaison VPN|
|**Port de service**|Communication avec le port de service|
|**Dynamique**|Connecte un VLAN à un WLAN|

### 🔗 Connectivité WLC

- Chaque interface logique = IP + masque + passerelle + serveur DHCP.
- Une interface logique est affectée à un **port physique** et un **ID VLAN**.
- Considérée comme une **terminaison de couche 3** sur un VLAN.
### 🧭 Méthodologie de dépannage (6 étapes)

|Étape|Titre|
|---|---|
|1|Identifier le problème|
|2|Élaborer une théorie des causes probables|
|3|Tester la théorie pour déterminer la cause|
|4|Élaborer un plan d'action et l'implémenter|
|5|Vérifier le fonctionnement global + mesures préventives|
|6|Documenter les résultats et actions|

# 📘 CHAPITRE 1 — PARTIE 5 : Comprendre le concept de routage

> ⏱️ Durée : 2h | 🎯 Objectif : Comprendre les concepts du routage et configurer les routes statiques

---

## 🗂️ Plan du chapitre

1. Concepts de routage
2. Dépanner les routes statiques et par défaut

---

## 1️⃣ Concepts de routage

### 🔧 Deux fonctions d'un routeur

1. **Déterminer le meilleur chemin** (via la table de routage)
2. **Transférer les paquets** vers leur destination

### 🎯 Détermination du chemin — la plus longue correspondance

- Le **meilleur chemin** = la route avec le **plus grand nombre de bits correspondants** à l'IP de destination (longest match).
- Exemple IPv4 : entre `172.16.0.0/12`, `/18` et `/26` → **`/26`** gagne (préfixe le plus long).
- Exemple IPv6 : entre `/40`, `/48` et `/64` → **`/48`** gagne (le `/64` ne correspond pas assez).

### 🗂️ Création de la table de routage

|Type de réseau|Comment il est ajouté|
|---|---|
|**Directement connecté**|Interface configurée avec IP + masque, et **active (up/up)**|
|**Distant — route statique**|Configuré **manuellement**|
|**Distant — protocole dynamique**|Appris **automatiquement**|
|**Route par défaut**|Utilisée quand aucune route spécifique ne correspond (préfixe **/0**) — dite "passerelle de dernier recours"|

### 📦 Transmission de paquets — processus de décision

1. Le paquet arrive sur l'interface d'entrée
2. Le routeur lit l'IP de destination et consulte sa table de routage
3. Il trouve le **préfixe le plus long correspondant**
4. Il encapsule le paquet et le transfère vers l'interface de sortie
5. Si **aucune correspondance** → le paquet est **supprimé**

Résultats possibles : transférer sur réseau connecté, transférer au saut suivant, ou **abandonner** le paquet.

### ⚙️ Mécanismes de transfert de paquets

|Mécanisme|Description|
|---|---|
|**Commutation de processus**|Ancien, traite **chaque paquet individuellement** via le CPU (lent)|
|**Commutation rapide**|Utilise un **cache** pour le saut suivant, évite de refaire le calcul pour chaque paquet|
|**CEF (Cisco Express Forwarding)**|Mécanisme **par défaut** actuel — construit une **FIB** et une table de contiguïté, mis à jour uniquement lors de changements de topologie|

### 📋 Table de routage IP — codes de source

|Code|Signification|
|---|---|
|**L**|Adresse assignée à une interface du routeur|
|**C**|Réseau directement connecté|
|**S**|Route statique|
|**O**|Réseau appris via **OSPF**|
|*****|Route candidate comme route par défaut|
### 🔎 Champs d'une entrée de table de routage

1. Source de la route
2. Réseau de destination (préfixe + longueur)
3. **Distance administrative** (fiabilité — plus bas = préféré)
4. **Métrique** (coût pour atteindre le réseau — plus bas = préféré)
5. Saut suivant
6. Horodatage
7. Interface de sortie

### 🌐 Route par défaut

- IPv4 : `0.0.0.0/0` | IPv6 : `::/0`
- Aucun bit ne doit correspondre → utilisée quand rien d'autre ne convient.

### 📏 Distance administrative (AD)

|Origine de la route|AD|
|---|---|
|Directement connecté|**0**|
|Route statique|**1**|
|Résumé de routage EIGRP|5|
|BGP externe|20|
|EIGRP interne|90|
|**OSPF**|**110**|
|IS-IS|115|
|RIP|120|
|EIGRP externe|170|
|BGP interne|200|

💡 Plus l'AD est **basse**, plus la route est **fiable**.

### ⚖️ Routage statique vs dynamique

|Critère|Dynamique|Statique|
|---|---|---|
|Complexité|Indépendante de la taille du réseau|Augmente avec la taille|
|Modifs de topologie|S'adapte automatiquement|Intervention manuelle requise|
|Extensibilité|Réseaux simples et complexes|Réseaux simples|
|Sécurité|À configurer|Inhérente|
|Ressources|CPU, mémoire, bande passante|Aucune ressource supplémentaire|
|Prévisibilité|Dépend de la topologie/protocole|Définie explicitement|

### 🧬 Classification des protocoles de routage

||IGP — Vecteur de distance|IGP — État de liens|EGP|
|---|---|---|---|
|**IPv4**|RIPv2, EIGRP|OSPFv2, IS-IS|BGP-4|
|**IPv6**|RIPng, EIGRP pour IPv6|OSPFv3, IS-IS pour IPv6|BGP-MP|

### 🎯 Objectifs des protocoles de routage dynamique

- Découverte des réseaux distants
- Mise à jour des infos de routage
- Choix du meilleur chemin
- Recherche d'un nouveau chemin si le chemin actuel échoue

### 📐 Métriques par protocole

|Protocole|Métrique|
|---|---|
|**RIP**|Nombre de sauts (max **15**)|
|**OSPF**|Coût basé sur la **bande passante cumulative** (liaisons rapides = coût faible)|
|**EIGRP**|Bande passante + délai les plus lents (+ fiabilité et charge en option)|

---

## 2️⃣ Dépanner les routes statiques et par défaut

### 🗂️ Types de routes statiques

- **Standard**
- **Par défaut**
- **Flottante**
- **Récapitulative (résumée)**

### 🔀 Options de tronçon suivant

|Type|Description|
|---|---|
|**Route de tronçon suivant**|Seule l'IP du saut suivant est spécifiée|
|**Route directement connectée**|Seule l'interface de sortie est spécifiée|
|**Route entièrement spécifiée**|IP du saut suivant **+** interface de sortie|

### 🛠️ Commandes de route statique

```
Router(config)# ip route <network-address> <subnet-mask> {<ip-address> | <exit-intf> [<ip-address>]} [distance]
Router(config)# ipv6 route <ipv6-prefix>/<prefix-length> {<ipv6-address> | <exit-intf> [<ipv6-address>]} [distance]
```

### 🔍 Vérification d'une route statique

- `show ip route static` / `show ipv6 route static`
- `show ip route <network>`
- `show running-config | section ip route`
- `ping`, `traceroute`

### 🌐 Route statique par défaut

```
Router(config)# ip route 0.0.0.0 0.0.0.0 {ip-address | exit-intf}
Router(config)# ipv6 route ::/0 {ipv6-address | exit-intf}
```

Utile pour connecter un routeur périphérique à un réseau fournisseur (routeur d'extrémité).

### 🛟 Route statique flottante

- Sert de **chemin de secours** (utilisée seulement si la route principale est indisponible).
- Configurée avec une **distance administrative plus élevée** que la route principale.

```
Router(config)# ip route 0.0.0.0 0.0.0.0 {ip-address} {distance}
Router(config)# ipv6 route ::/0 {ipv6-address} {distance}
```

### 🎯 Route statique d'hôte

- Cible un **périphérique spécifique** (masque `/32` en IPv4, `/128` en IPv6).

```
Branch(config)# ip route 209.165.200.238 255.255.255.255 198.51.100.2
Branch(config)# ipv6 route 2001:db8:acad:2::238/128 2001:db8:acad:1::2
```

### 🧰 Dépannage — causes fréquentes d'échec réseau

- Interface désactivée
- Perte de connexion FAI
- Liaisons saturées
- Erreur de configuration

### 🔍 Commandes de dépannage

|Commande|Description|
|---|---|
|`ping`|Vérifie la connectivité couche 3|
|`traceroute`|Vérifie le chemin (via ICMP)|
|`show ip route`|Affiche la table de routage|
|`show ip interface brief`|État et IP des interfaces|
|`show cdp neighbors`|Périphériques Cisco directement connectés (couches 1 et 2)|

# stop
# 🌐 OSPF 

## 1. 🧠 Concepts essentiels

- **OSPF** (_Open Shortest Path First_) : protocole de routage à état de liens.
- ⚡ Convergence rapide, adapté aux grands réseaux.
- 🧮 **SPF (Dijkstra)** calcule le meilleur chemin selon le coût cumulé : **coût faible = meilleur chemin**.
- 🌍 **OSPFv2** : IPv4 · **OSPFv3** : IPv6.
- 🦴 **Area 0** : backbone. En multi-area, les autres zones doivent se connecter à Area 0 via un **ABR**.

## 2. 📦 Paquets OSPF

|Type|Nom|Rôle|
|---|---|---|
|1|👋 Hello|Découvrir les voisins|
|2|📋 DBD|Résumer/synchroniser les bases de données|
|3|🔎 LSR|Demander des informations de liens|
|4|📤 LSU|Envoyer les informations demandées (LSA)|
|5|✅ LSAck|Accuser réception|

💡 **LSU contient des LSA.**

## 3. 🗂️ Tables OSPF

- 🤝 **Voisins / adjacency** : routeurs voisins — `show ip ospf neighbor`
- 🗃️ **LSDB / topologie** : état des liens; identique entre routeurs d'une même zone — `show ip ospf database`
- 🧭 **Table de routage** : meilleurs chemins calculés — `show ip route`

## 4. 🔄 États des voisins

`Down → Init → Two-Way → ExStart → Exchange → Loading → Full`

- ↔️ **Two-Way** : communication bidirectionnelle; élection DR/BDR possible sur Ethernet.
- 🎯 **Full** : LSDB synchronisée.
- 💡 Sur Ethernet multi-accès, **2-Way entre deux DROTHERs peut être normal**.

## 5. 👑 DR / BDR

- Réduisent les adjacences et la diffusion des LSA sur les réseaux multi-accès.
- 🏆 Élection : priorité d'interface la plus élevée, puis Router ID le plus élevé.
- ⚙️ Priorité par défaut : `1`; priorité `0` = jamais DR/BDR.
- 🛟 Le BDR remplace le DR en cas de panne.

```
interface g0/0/0
 ip ospf priority 255
```

🔍 Vérifier : `show ip ospf neighbor`

## 6. ⚙️ Configuration OSPFv2 — zone unique

### 🧩 Méthode `network`

```
router ospf 10
 router-id 1.1.1.1
 network 10.10.1.0 0.0.0.255 area 0
 network 10.1.1.4 0.0.0.3 area 0
```

💡 Le **wildcard mask** est généralement l'inverse du masque de sous-réseau.

### 🔌 Activer OSPF directement sur une interface

```
interface g0/0/0
 ip ospf 10 area 0
```

### 🔇 Interface passive

Annonce le réseau, mais n'envoie pas de Hello sur cette interface :

```
router ospf 10
 passive-interface loopback 0
```

### 🔗 Type point-à-point

Désactive l'élection DR/BDR sur l'interface concernée :

```
interface g0/0/0
 ip ospf network point-to-point
```

### 🆔 Router ID

```
router ospf 10
 router-id 1.1.1.1
```

Après modification, redémarrer le processus pour appliquer le changement :

```
clear ip ospf process
```

## 7. 🧮 Coût OSPF et minuteries

- 📐 Formule Cisco : `coût = bande passante de référence / bande passante de l'interface`.
- ⚠️ Référence par défaut : **100 Mbps**; le coût plus faible est préféré.
- 🚀 Pour les liens rapides, ajuster la référence sur tous les routeurs :

```
router ospf 10
 auto-cost reference-bandwidth 10000
```

- 🛠️ Coût manuel :

```
interface g0/0/1
 ip ospf cost 30
```

- ⏱️ Valeurs OSPFv2 par défaut sur Ethernet/point-à-point : **Hello 10 s**, **Dead 40 s**. Les minuteries doivent correspondre entre voisins.

```
interface g0/0/1
 ip ospf hello-interval 5
 ip ospf dead-interval 20
```

## 8. 🌐 Propager une route par défaut

Le routeur périphérique doit avoir une route par défaut puis l'annoncer :

```
ip route 0.0.0.0 0.0.0.0 <next-hop>
router ospf 10
 default-information originate
```

## 9. 🌍 OSPFv3 — IPv6

- Activer le routage IPv6 :

```
ipv6 unicast-routing
```

- Créer le processus et configurer le Router ID :

```
ipv6 router ospf 10
 router-id 1.1.1.1
```

- Activer OSPFv3 sur chaque interface :

```
interface g0/0/0
 ipv6 ospf 10 area 0
```

- 🔗 OSPFv3 utilise des adresses link-local IPv6 pour les échanges entre voisins.
- 📡 Multicast : `FF02::5` (tous les routeurs OSPF), `FF02::6` (DR/BDR).

## 10. 🔍 Vérification — commandes à retenir

### OSPFv2

```
show ip protocols
show ip ospf
show ip ospf interface
show ip ospf interface brief
show ip ospf neighbor
show ip ospf database
show ip route ospf
```

### OSPFv3

```
show ipv6 protocols
show ipv6 ospf
show ipv6 ospf interface
show ipv6 ospf interface brief
show ipv6 ospf neighbor
show ipv6 ospf database
show ipv6 route ospf
```

## ⚡ Mémo express

- 👋 **Hello** = découvre les voisins.
- ✅ **Full** = bases de données synchronisées.
- 🦴 **Area 0** = backbone.
- 👑 **DR/BDR** = réseaux multi-accès.
- 🧮 **Coût faible** = meilleur chemin.
- 🌍 **OSPFv2 = IPv4** · **OSPFv3 = IPv6**.

# ACL
## 1.ACL

ACL : un list de regle qui configurer dans le reouteur pour 
- Contrôler le trafic entrant et sortant.
- Améliorer la sécurité du réseau.
## 2. Types d’ACL

| Type         | Fonction                                                                     | numuro          |
| ------------ | ---------------------------------------------------------------------------- | --------------- |
| ACL standard | Filtre principalement selon l’adresse IP source.                             | 1–99, 1300–1999 |
| ACL étendue  | Peut filtrer selon l’IP source, l’IP destination, le protocole et les ports. | 1–99, 1300–1999 |

## 3. Configuration

### ACL numérotée standard

```
R1(config)#access-list 1 permit 192.168.0.0 0.0.0.255
R1(config)#access-list 1 permit 192.168.1.0 0.0.0.255
R1(config)#access-list 1 deny 192.168.0.0 0.0.3.255
R1(config)#access-list 1 permit any
```

### ACL nommée standard

```
R1(config)#ip access-list standard monACL
R1(config-std-nacl)#permit 192.168.0.0 0.0.0.255
R1(config-std-nacl)#permit 192.168.1.0 0.0.0.255
R1(config-std-nacl)#deny 192.168.0.0 0.0.3.255
R1(config-std-nacl)#permit any
```

> Les deux sont identiques : tout le trafic de 192.168.0.0/22 est bloqué, sauf les sous-réseaux 192.168.0.0/24 et 192.168.1.0/24.

### ACL numérotée étendue

```
R1(config)#access-list 100 permit tcp any host 192.168.1.100 eq 80
R1(config)#access-list 100 permit icmp 192.168.0.0 0.0.0.255 host 192.168.1.100
```

### ACL nommée étendue

```
R1(config)#ip access-list extended monACLextended
R1(config-ext-nacl)#permit tcp any host 192.168.1.100 eq 80
R1(config-ext-nacl)#permit icmp 192.168.0.0 0.0.0.255 host 192.168.1.100
```

> Effet : HTTP vers 192.168.1.100 autorisé ; ICMP de 192.168.0.0/24 vers 192.168.1.100 autorisé ; tout le reste rejeté.

### Format général d'une règle étendue

```
<action> <protocole> <IP source> [port source] <IP dest> [port dest] [options]
```

- **action** : `permit` / `deny`
- **protocole** : `tcp`, `udp`, `icmp`… ; `ip` = tous les protocoles
- **adresse** : adresse + wildcard mask, ou `host x.x.x.x`, ou `any` (n'importe laquelle)

## 6. Vérification

```
R1#show access-lists
```

Affiche les règles avec leur numéro de séquence (10, 20, 30…).

## 7. Modifier une ACL

```
R1(config)#ip access-list standard 1
R1(config-std-nacl)#no 20                          ! supprime la règle n° 20
R1(config-std-nacl)#15 permit 192.168.1.0 0.0.0.127 ! ajoute une règle en position 15
```

## 8. Supprimer une ACL

```
R1(config)#no access-list 100                  ! ACL numérotée
R1(config)#no ip access-list standard monACL   ! ACL nommée
```

## 9. Appliquer une ACL

### Sur une interface

```
R1(config)#interface fastethernet 0/0
R1(config-if)#ip access-group 1 in     ! trafic entrant
R1(config-if)#ip access-group 1 out    ! trafic sortant
```

Désactiver : `no ip access-group 1 in|out`

Vérifier : `show ip interface fastEthernet 0/0` (affiche _Inbound / Outgoing access list_).
# 🌐 NAT & CDP/LLDP – Short Summary

---
## 🔄 NAT

Translates private IPs (RFC 1918) to public IPs.

**📖 Terms:** Inside local (private) | Inside global (public) | Outside global (real remote) | Outside local (remote as seen inside)

### 🧩 Types

- 📌 **Static**: 1-to-1 fixed mapping (servers)
- 🎯 **Dynamic**: private IPs → pool of public IPs
- 🔀 **PAT (Overload)**: many private → 1 public IP using ports

### ✅ Advantages

- 💰 Saves public IPv4
- 🙈 Hides internal network
- 🔧 No renumbering when changing ISP

### ❌ Disadvantages

- Complexité de configuration.
- Problèmes avec certaines applications.
- Diminution de la vitesse du réseau.
### ⚙️ Configuration (Cisco)

```
interface g0/0
 ip nat inside
interface g0/1
 ip nat outside

! Static
ip nat inside source static 192.168.1.10 203.0.113.10

! Dynamic
access-list 1 permit 192.168.1.0 0.0.0.255
ip nat pool MYPOOL 203.0.113.10 203.0.113.20 netmask 255.255.255.0
ip nat inside source list 1 pool MYPOOL

! PAT
ip nat inside source list 1 interface g0/1 overload
```

**🔎 Verify:** `show ip nat translations` | `show ip nat statistics` | `clear ip nat translation *`

### ⚖️ NAT vs PAT vs NAT64

||🔄 NAT|🔀 PAT|🆕 NAT64|
|---|---|---|---|
|Translation|IP ↔ IP|IP:port ↔ IP:port|IPv6 ↔ IPv4|
|Purpose|Private/public|Save public IPs|IPv6-only clients reach IPv4 servers|

💡 NAT64 works with DNS64, prefix `64:ff9b::/96`.

---
## 📡 CDP vs LLDP

**CDP (Cisco Discovery Protocol) :** Permet de découvrir les équipements Cisco directement connectés au réseau et de partager leurs informations (nom, adresse IP, port connecté, etc.).

Layer 2 neighbor discovery protocols.

||🟦 CDP|🟩 LLDP|
|---|---|---|
|Standard|Cisco proprietary|IEEE 802.1AB|
|Multicast MAC|01:00:0C:CC:CC:CC|01:80:C2:00:00:0E|
|Timers|60 s / hold 180 s|30 s / hold 120 s|
|Default (Cisco)|✅ Enabled|❌ Disabled|
|Use|Cisco-only|Multi-vendor|

### 🟦 CDP config

```
cdp run
interface g0/1
 no cdp enable
cdp timer 30
cdp holdtime 120
```

**🔎 Verify:** `show cdp neighbors [detail]`

### 🟩 LLDP config

```
lldp run
interface g0/1
 lldp transmit
 lldp receive
lldp timer 30
lldp holdtime 120
```

**🔎 Verify:** `show lldp neighbors [detail]`

🔒 **Security:** disable both on untrusted ports.
# VOIP CUCME

```bash
! 1. DHCP
ip dhcp pool VOIP
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 option 150 ip 192.168.10.1
 exit

! 2. CUCME
telephony-service
 max-ephones 2
 max-dn 2
 ip source-address 192.168.10.1 port 2000
 auto assign 1 to 2
 exit

! 3. Telephone extensions
ephone-dn 1
 number 1001
 exit

ephone-dn 2
 number 1002
 exit

! 4. Generate configuration files
telephony-service
 create cnf-files
end
```