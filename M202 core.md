#### 🌐 IP statique
## 📂 Résumé : Groupes et Autorisations Active Directory

#### 👥 1. Types de Groupes
* **📧 Groupes de distribution :** Pour la messagerie uniquement. Sans sécurité (pas de SID), aucune autorisation possible.
* **🛡️ Groupes de sécurité :** Pour la gestion des accès (avec SID). Peuvent aussi servir pour la messagerie.
#### 🌐 2. Étendue des Groupes (Scopes)

| 🔹 **Étendue**       | 👥 **Membres**    | 🔐 **Autorisations**                          |
| -------------------- | ----------------- | --------------------------------------------- |
| 🖥️ **Local**        | U, O, GG, GLD, GU | 💻 Ordinateur local uniquement                |
| 🏢 **Domaine local** | U, O, GG, GU      | 🌐 N’importe où ( في أي مكان) dans le domaine |
| 🌍 **Universel**     | U, O, GG, GU      | 🌐 N’importe où dans la forêt                 |
| 🌐 **Global**        | U, O, GG          | 🏢 Domaine ou domaine approuvé                |
U 👤 = Utilisateur | O 💻 = Ordinateur | GG 👥 = Groupe global | GLD 🔵 = Groupe local de domaine | GU 🌍 = Groupe universel

![[gro.jpeg]]

### 🔐 AGDLP — simplement

**AGDLP = Account → Global → Domain Local → Permission**

1. 👤 **A – Account** → Les utilisateurs.
2. 👥 **G – Global Group** → Mettre les utilisateurs dans un groupe global.
3. 🔵 **DL – Domain Local** → Ajouter le groupe global au groupe local du domaine.
4. 🔑 **P – Permission** → Donner les droits au groupe local.
#### Exemple
**Ali 👤 → Sales 👥 → ACL_Sales_Read 🔵 → Lecture 📖**
# 🌳 1. Active Directory

- 🌐 **Domaine** → créé sur le contrôleur de domaine principal.
- 👤 **Utilisateurs** → création et gestion des comptes.
- 📁 **OU (Unité d’Organisation)** → organisation des utilisateurs et objets.
- 🖥️ **Contrôleur secondaire** → peut être modifiable.

---

## 🖥️ 2. Contrôleur de domaine secondaire

🎯 **Objectif :** assurer la **redondance** et la **haute disponibilité**.

- 🔄 Redondance
- 🔐 Authentification en cas de panne
- 🌐 Plusieurs contrôleurs dans le réseau

### ✏️ Contrôleur modifiable

**Lecture + création + modification + suppression** des objets.

### 👀 RODC

**Lecture seule** 🔒  
➡️ Utile notamment lorsque la sécurité doit être renforcée.

---

## 🔄 3. Réplication

**Réplication = synchronisation des informations entre les contrôleurs de domaine.**
## type Réplication
### 🏢 Intra-site

- Même site / réseau
- ⚡ Généralement rapide
### 🌍 Inter-site

- Sites / réseaux différents
- 🐢 Généralement plus lente

---
## 👤 4. Utilisateur de domaine

- 👤 Un utilisateur de domaine peut appartenir à des groupes.
- ⚠️ **Utilisateur de domaine ≠ Administrateur**
- 🔐 La connexion locale à un serveur dépend des droits/politiques.

---
## 🔑 5. Mot de passe

⚠️ L’utilisateur peut être obligé de **modifier son mot de passe avant la première connexion**.

---
## 💿 6. Installation

Sources possibles :

- 💿 DVD
- 🔑 Clé USB
- 💾 Disque dur externe

---
## 🌐 7. Configuration réseau

Pour les machines :

1. 📍 Configurer l’**adresse IP**
2. 🧩 Vérifier le **masque**
3. 🌐 Configurer le **DNS**
4. 🔌 Tester la **connectivité**

---
### 🧠 À retenir

> 🌳 **AD** → Utilisateurs + OU  
> 🖥️ **Contrôleur secondaire** → Redondance + disponibilité  
> ✏️ **Modifiable** → Lecture + modification des objets  
> 👀 **RODC** → Lecture seule  
> 🔄 **Réplication** → Synchronisation  
> 👤 **Utilisateur domaine** ≠ Administrateur  
> 🔌 **Réseau** → IP + Masque + DNS + Connectivité

### 🖥️ Installation DC additionnel

1. 🌐 Configurer **IP + DNS**
2. 🔗 Joindre le **domaine**
3. 🖥️ Installer **AD DS**
4. 🔄 **Promote this server to a domain controller**
5. ➕ **Add a domain controller to an existing domain**
6. 🔐 Définir le **DSRM password**
7. ✅ **Install → Redémarrer**
### 🌳 Installation d’un sous-domaine

1. 🌐 Configurer **IP + DNS**
2. 🖥️ Installer **AD DS**
3. 🔄 **Promote this server to a domain controller**
4. ➕ Choisir **Add a new domain to an existing forest**
5. 🌳 Sélectionner **Child Domain**
6. ✏️ Entrer le nom du sous-domaine
7. 🔐 Entrer les identifiants du domaine parent
8. ⚙️ **Install → Redémarrer**

## 🔐 Gestion des permissions avec AGDLP

**AGDLP** est une méthode pour organiser les utilisateurs et les permissions dans Active Directory.
👉 **U → GG → DL → Permission**
- 👤 **U = User** → utilisateur
- 👥 **GG = Global Group** → groupe global qui contient les utilisateurs
- 📁 **DL = Domain Local Group** → groupe qui reçoit les permissions
- 🔐 **Permission** → droit sur un dossier
### 📌 Exemple
Ali et Sara travaillent en **Informatique** :

**Ali + Sara**  
⬇️  
**GG_Informatique**  
⬇️  
**DL_Dossier_Informatique_Modification**  
⬇️  
📁 `C:\Entreprise\Informatique`  
⬇️  
✏️ **Modification**

## 🖥️ FSMO

| Rôle                          | Fonction simple                                          | ❌ Si le rôle est perdu                        |
| ----------------------------- | -------------------------------------------------------- | --------------------------------------------- |
| 🧩 **Maître de schéma**       | Gère le **schéma Active Directory**.                     | Modification du schéma impossible             |
| 🌐 **Maître de dénomination** | Vérifie que le **nom du domaine est unique**.            | Ajout/suppression de domaine impossible       |
| 🆔 **Maître RID**             | Attribue les **RID** pour créer les **SID**              | Création de nouveaux objets peut échouer      |
| 🔗 **Maître Infrastructure**  | Vérifie et met à jour les **objets des autres domaines** | Problèmes avec les objets des autres domaines |
| ⏰ **PDC Emulator**            | Synchronise le **temps** == Horloge synchronisée         | Problèmes de temps et de mots de passe        |

- **2 rôles = forêt** → Schéma + Dénomination
- **3 rôles = domaine** → RID + Infrastructure + PDC Emulator
---

🆔 **RID** → Numéro unique pour chaque **objet** du domaine.
### Changer FSMO GUI

- 🖥️ Ouvrir **AD Users and Computers**
- 🖱️ **Domain → Operations Masters**
- 🔄 **Change** → sélectionner **CD2**
- 🌳 **Domain Naming Master** → _Domains and Trusts → Operations Master → Change_
- 🏢 **Schema Master** → _Active Directory Schema → Operations Master → Change_
#### 🏢 etaps for add Activer **Active Directory Schema**

1. 🖥️ **Server Manager** → **Tools**
2. 🔧 **Active Directory Schema**
3. Si absent : **Tools → Register `schmmgmt.dll`**
4. Ouvrir **MMC** → **Add/Remove Snap-in**
5. Ajouter **Active Directory Schema** ✅

---
### Changer FSMO CLI

### 🔄 Changer FSMO — CLI

**Vérifier :**

```shell
netdom query fsmo
```

**Transférer les 5 rôles vers CD2** :

```shell
Move-ADDirectoryServerOperationMasterRole -Identity "CD2" -OperationMasterRole 0,1,2,3,4
```

- 🆔 **RID Master** → `0`
- ⏰ **PDC Emulator** → `1`
- 🔗 **Infrastructure Master** → `2`
- 🌳 **Schema Master** → `3`
- 🏢 **Domain Naming Master** → `4`

🌐 **Global Catalog (GC)** → Facilite la **recherche et l’authentification** dans la forêt.
💿 **IFM (Install From Media)** → Installe un **contrôleur de domaine** à partir d’un **support préparé**.

# 🖥️ Gestion des ordinateurs dans un domaine Active Directory

### 👑 1. Contrôleur de domaine

- DC : gère le domaine et l’authentification.
    
- Serveur membre : appartient au domaine sans être DC.
    
- 🔄 Rétrogradation : transformer un DC en serveur normal.
### 🔗 2. Joindre un ordinateur au domaine

- Nécessite le nom du domaine et un compte autorisé.
    
- 💻 Crée ou utilise un objet ordinateur dans Active Directory.
### 📂 3. Objets ordinateur, OU et GPO

- Computers : conteneur par défaut des ordinateurs.
    
- OU : organise les objets ordinateur.
    
- 📜 GPO : applique des stratégies aux ordinateurs et utilisateurs.
### 🔐 4. Compte ordinateur

- 🚫 Désactiver un compte ordinateur : bloquer temporairement son accès au domaine sans supprimer son objet AD.
    
- 🔄 Réinitialiser : utile en cas de problème de relation avec le domaine.
### 🛡️ 5. Canal sécurisé

- Assure la communication sécurisée entre le client et le DC.
    
- ❌ Un problème de mot de passe peut bloquer l’authentification.
    
- 🔧 Vérifier le compte ordinateur et réparer le canal si nécessaire.
### ⌨️ 6. LDAP

==> un protocole qui permet d'accéder aux informations et de gérer les objets stockés dans un annuaire

- `CN` → Nom de l’objet
    
- `OU` → Unité d’organisation
    
- `DC` → Composant du domaine
    
- Exemple : `CN=PC,OU=Clients,DC=exemple,DC=local`


# 📂 DHCP (go to your book and reviser)
- Base de données DHCP : `C:\Windows\System32\dhcp\`

| Option              | Fonction                                         |
| ------------------- | ------------------------------------------------ |
| Routeur (Router)    | Définit la passerelle par défaut.                |
| Serveurs DNS        | Définit les serveurs DNS à utiliser.             |
| Nom de domaine DNS  | Définit le nom du domaine DNS.                   |
| Serveurs NTP        | Définit les serveurs de synchronisation horaire. |
| Durée du bail       | Définit la durée de validité de l’adresse IP.    |
| Nom du serveur TFTP | Indique le serveur TFTP.                         |

### 📡 Haute disponibilité DHCP

##### 1. ⚖️ Équilibrage de charge (Load Balance)

- Deux serveurs DHCP actifs.
    
- Les requêtes sont réparties entre les deux.
    
- ✅ Avantage : meilleure répartition de la charge.
#### 2. 🛡️ Serveur de secours (Hot Standby)

- Un serveur principal et un serveur secondaire en attente.
    
- Le secondaire prend le relais en cas de panne.
    
- ✅ Avantage : continuité du service.

#### 3. 🔀 Fractionnement d’étendue (DHCP Split Scope)

- Une étendue DHCP est divisée entre deux serveurs.
    
- Chaque serveur distribue une partie des adresses IP.
    
- ✅ Avantage : continuité partielle du service en cas de panne.

💡 À retenir :

- Load Balance = partager la charge.
    
- Hot Standby = serveur de secours.
    
- Split Scope = partager la plage IP.
# 📂 DNS (go to your book and reviser)
- Fichier de configuration DNS principal : `C:\Windows\System32\dns\

# 🛡️ Stratégies de groupe (GPO)

## 1️⃣ GPO — Créer, appliquer et filtrer

### 📌 Qu'est-ce qu'un GPO ?

Un **GPO (Group Policy Object)** est un ensemble de **paramètres** appliqués automatiquement aux **utilisateurs** et aux **ordinateurs** dans Active Directory.
### 🔗 Ordre d'application (LSDOU)

**L**ocal → **S**ite → **D**omaine → **O**U   != if we use aplique 

> ⚠️ Le dernier appliqué l'emporte (sauf si un GPO est **Appliqué/Enforced**) ,can use heritage betwen layer.
### eample
```css
1. Local → Blue

2. Site → Green

3. Domain → Yellow

4. OU → Red
```
#### resulte: Red 🔴

### GPO – LSDOU & Exceptions

| لحالة             | التأثير                            |
| ----------------- | ---------------------------------- |
| Normal LSDOU      | الأخيرة غالباً كتربح               |
| Enforced          | كتقدر تحافظ على أسبقية GPO الأصلية |
| Block Inheritance | كتمنع الوراثة                      |
| Link Order 1      | أعلى أسبقية فنفس OU                |
### ✅ Créer et appliquer un GPO (étapes)

1. Tools → Group Policy Management.
2. Clic droit sur le domaine/OU → Create a GPO.
3. Donner un nom → OK.
4. Clic droit sur le GPO → Edit.
5. Configurer la stratégie → Enabled/Disabled.
6. Sur le client, exécuter :

```shel
gpupdate /force
```

7. Vérifier :

```shell
gpresult /r
```
### 🎯 Filtrage

|🧰 Méthode|📖 Explication|
|---|---|
|🔐 **Filtrage de sécurité**|Par défaut, le GPO cible _Utilisateurs authentifiés_. On le remplace par un groupe précis (droits **Lecture + Appliquer la stratégie de groupe**)|
|🧠 **Filtre WMI**|N'applique le GPO que si une condition est vraie (ex. version de Windows)|
|🚫 **Bloquer l'héritage**|Une OU ignore les GPO des niveaux supérieurs|
|💪 **Appliqué (Enforced)**|Force le GPO, même si l'héritage est bloqué|
|🔒 **Ciblage au niveau de l'élément**|(Préférences) filtrage fin par paramètre|
**Exemple de filtre WMI** :

```sql
SELECT * FROM Win32_OperatingSystem WHERE Version LIKE "10.%" AND ProductType = "1"
```

---

# here

## 2️⃣ PSO — Stratégies de mot de passe affinées (Fine-Grained)

### 📌 Qu'est-ce qu'un PSO ?

Objet Active Directory permettant d'appliquer des stratégies de mots de passe spécifiques à certains utilisateurs ou groupes
### ⚙️ Paramètres principaux

| Paramètre                             | Signification                               |
| ------------------------------------- | ------------------------------------------- |
| 🔢 `MinPasswordLength`                | Longueur minimale                           |
| 🧬 `PasswordComplexityEnabled`        | Complexité (maj., min., chiffres, symboles) |
| ⏳ `MaxPasswordAge` / `MinPasswordAge` | Durée de vie du mot de passe                |
| 📜 `PasswordHistoryCount`             | Nombre d'anciens mots de passe mémorisés    |
| 🚪 `LockoutThreshold`                 | Nombre d'échecs avant verrouillage          |
| ⏱️ `LockoutDuration`                  | Durée du verrouillage                       |
| 🥇 `Precedence`                       | Priorité (le plus petit nombre l'emporte)   |
### 📋 Règles importantes

- 🎯 S'applique aux **utilisateurs** et **groupes de sécurité globaux** (pas directement aux OU)
- 🥇 Si plusieurs PSO : la **Precedence** la plus basse gagne
- 🏗️ Niveau fonctionnel du domaine ≥ **Windows Server 2008**

### 💻 Créer un PSO avec PowerShell

```powershell
# Création du PSO
New-ADFineGrainedPasswordPolicy -Name "PSO_Admins" `
  -Precedence 10 `
  -MinPasswordLength 14 `
  -PasswordHistoryCount 24 `
  -ComplexityEnabled $true `
  -LockoutThreshold 3 `
  -LockoutDuration "00:30:00" `
  -LockoutObservationWindow "00:30:00" `
  -MaxPasswordAge "30.00:00:00" `
  -MinPasswordAge "1.00:00:00"

# Application à un groupe
Add-ADFineGrainedPasswordPolicySubject -Identity "PSO_Admins" -Subjects "Domain Admins"

# Vérification
Get-ADFineGrainedPasswordPolicy -Filter *
Get-ADUserResultantPasswordPolicy -Identity "user1"
```

> 🖱️ Via l'interface : **ADAC** (Centre d'administration Active Directory) → `Domaine` → `System` → `Password Settings Container` → **Nouveau → Paramètres de mot de passe**

---

## 3️⃣ Outils d'administration — GPMC et PowerShell

### 🖥️ GPMC (Console de gestion des stratégies de groupe)

|✨ Fonction|📖 Usage|
|---|---|
|📂 Créer / lier / supprimer un GPO|Gestion complète|
|💾 **Sauvegarde / Restauration**|Copie de sécurité des GPO|
|📊 **Modélisation de stratégie de groupe**|Simulation avant application|
|🔎 **Résultats de stratégie de groupe**|Ce qui est réellement appliqué à un utilisateur/PC|
|📤 Importer / Copier un GPO|Transfert entre domaines|
### 🛠️ Commandes utiles (CMD)

|Commande|Rôle|
|---|---|
|`gpupdate /force`|🔄 Actualisation immédiate|
|`gpresult /r`|📊 Résumé des stratégies appliquées|
|`gpresult /h rapport.html`|📄 Rapport HTML|
|`rsop.msc`|🔍 Jeu de stratégie résultant|

---
# 💽 Disques, Volumes, RAID & Styles de partition
---
## 💾 Types de disques (Windows)

- **Disque de base** : utilise des partitions (MBR : 4 primaires max). Simple, pas de RAID.
- **Disque dynamique** : utilise des volumes. Gère les volumes fractionnés, agrégés par bandes, en miroir et RAID-5. Le retour vers disque de base efface les données.

- **Disque de base**  ---> **Disque dynamique** (perdree le dual boot)
- **Disque dynamique**  ---> **Disque de base** (impossible sans perte de donnees)

---
## Advantages and disadvantages
#### MBR
- Easy compatibility with old computers.
    
- Maximum 4 primary partitions.
    
- Limited to approximately 2 TB with 512-byte sectors.
#### GPT
- Supports large hard disks (more than 2 TB).
    
- Supports up to 128 partitions in Windows.
    
- Recommended for modern computers using UEFI.
---

## 💽 Types de volumes – Windows

## 1. Simple Volume

![Simple Volume diagram](https://images.openai.com/static-rsc-4/_dt_8jk1LBRUqMc3-LydegaSURuP9QlX3WK5yEOsXbOUUw0kp-19lomlJ2xFgT4rWxlEtPVIjUEkpJEvHlwrcPXmLqIpoo-UXlAYSu8oob8x5JLX2colXsXwkdsgMXcckFYtdsmRbDLze94u0C5kozDbF0NDJ2DAQRQFA8BMfmg?purpose=inline)

- Uses only **one disk**.
- Normal data storage.
- No data protection.

**Example:** Disk 1 = 100 GB → Volume = 100 GB.

---

## 2. Spanned Volume

![Spanned Volume diagram](https://images.openai.com/static-rsc-4/p6nmorPGdUqHXGU0y8WwGcc7SeVWIdP2ke7FSWwKMtnkPSzPQiMoi5rWvcrL2SZga--Hq4R_5F63lt8nWFRR-CGuO1VEiJT3zQMct3vEA24-XZJPqxz7djV8DF9OTcW8ybtWKc2Ip_8Nh3ddNFgAq-6LyOIohtSYYpBb8xTDvrk?purpose=inline)

- Combines space from **2 or more disks**.
- Creates one large volume.
- No data protection.

**Example:** 100 GB + 200 GB = 300 GB.

---

## 3. Striped Volume (RAID 0)

![Striped Volume RAID 0 diagram](https://images.openai.com/static-rsc-4/WNyIxusFCwMTdefsPgBDAgrR5uxeG0IS5Lhf6O1MleeCJsi1Nyivf7k-VglbyLY0h33KuHshWMmKLTedk0LaHoFOa3SU6-VtIW4dRfuNfTPwiwsH8jeBM9VgMGNuG1WLwnzH3t_S7YDH-aO-RTxqK1rW7vfES8r30zr_n33iGPU?purpose=inline)

- Requires at least **2 disks**.
- Distributes data across disks.
- Increases performance (speed).
- No data protection.

**Example:** 2 × 100 GB = 200 GB.

---

## 4. Mirrored Volume (RAID 1)

![Mirrored Volume RAID 1 diagram](https://images.openai.com/static-rsc-4/NNOsRDQyBxqCC9TRkmnphFxSfLLBbM_w2kU_ZCZyI-kRWnlq1v5udNR8hDeMEKeVkWHacgbXVqh6cOEZNXyOiLj8Y7VA7L_aHg_RQncQCc8ScWpnmW0s0UOa2FvIi2Lm5BPYiEl8cPalqYp341d-OAeKTAuIKImxOY0HRTlKtMA?purpose=inline)

- Requires **2 disks**.
- Copies the same data to both disks.
- Protects against one disk failure.
- Usable capacity = 50% with two equal disks.

**Example:** 2 × 100 GB = 100 GB usable.

---

## 5. RAID-5 Volume

![RAID 5 Volume diagram](https://images.openai.com/static-rsc-4/CJ_sXgf9nIoljErAq7EMXDgSYYbtueVS98j7cX-BDyFk3P44t01LIINimsZ4iVu96cot7eYavim0q8BGOcxQ4R2K_schGJeAjLnmSkzRc9YEvgfaQ4eN0So-_s6ks1InBfmsnOxy7G4ZROfGS5W7pHZV0i-m4IwILmx8ko1xQy4?purpose=inline)

- Requires at least **3 disks**.
- Uses data striping + distributed parity.
- Can tolerate **one disk failure**.
- Good balance between storage and protection.

**Example:** 3 × 100 GB = 200 GB usable.

---
## Quick Revision 🧠

| Volume type | Minimum disks | Main purpose | Fault tolerance |
|---|---:|---|---|
| Simple | 1 | Normal storage | No |
| Spanned | 2 | Combine disk space | No |
| Striped (RAID 0) | 2 | Performance / speed | No |
| Mirrored (RAID 1) | 2 | Duplicate data | Yes, one disk |
| RAID-5 | 3 | Distributed parity | Yes, one disk |

## 1. FSRM (File Server Resource Manager)
## 2. Quotas
Un **quota** permet de limiter ou de surveiller l'espace disque utilisé  
par un utilisateur ou un dossier.
#### Quota non conditionnel (Hard Quota)
- Fixe une limite stricte.
- Empêche l'utilisateur de dépasser la limite configurée.
### Quota conditionnel (Soft Quota)
- Définit un seuil de surveillance.
- L'utilisateur peut dépasser le seuil.

## 3. Filtrage des fichiers (File Screening)
Le filtrage des fichiers permet de contrôler les types de fichiers  
enregistrés dans un dossier.

### Filtrage actif (Active File Screening)
- Bloque l'enregistrement des types de fichiers interdits.
- Exemple : empêcher les utilisateurs d'enregistrer des fichiers  
    `.mp3`.

### Filtrage passif (Passive File Screening)
- Surveille les fichiers sans empêcher leur enregistrement.
- Peut générer des notifications ou des rapports lorsqu'un fichier  
    interdit est détecté.