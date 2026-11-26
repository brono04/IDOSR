# M202 — Administrer un environnement Windows
## Fiche de révision OFPPT

> **Objectif :** préparer la révision des compétences DHCP, DNS, GPO/PSO, serveur de fichiers/FSRM, DFS et WDS.
>
> **Règle de lecture**
> - **📘 Au programme PDF** : notions et tâches explicitement indiquées dans le support OFPPT.
> - **💻 Complément pratique** : commandes et exemples PowerShell ajoutés pour faciliter les travaux pratiques. Ils ne sont pas tous explicitement exigés par le PDF.
> - Remplacer les noms, domaines, IP et chemins d'exemple par ceux du laboratoire.

---
## Sommaire

1. [Vue d'ensemble du programme](#1-vue-densemble-du-programme)
2. [DHCP](#2-dhcp)
3. [DNS](#3-dns)
4. [GPO et PSO](#4-gpo-et-pso)
5. [Serveur de fichiers et FSRM](#5-serveur-de-fichiers-et-fsrm)
6. [DFS](#6-dfs--distributed-file-system)
7. [WDS](#7-wds--windows-deployment-services)
8. [Méthode pour les travaux pratiques](#8-méthode-pour-les-travaux-pratiques)
9. [Questions de révision](#9-questions-de-révision)
10. [Checklist avant l'examen](#10-checklist-avant-lexamen)
11. [Références au programme PDF](#11-références-au-programme-pdf)

---
# 1. Vue d'ensemble du programme

| Technologie | Rôle principal | Ce qu'il faut savoir faire |
|---|---|---|
| DHCP | Attribution automatique de la configuration IP | Installer, créer une étendue, configurer les options, réserver un bail, sauvegarder/restaurer |
| DNS | Résolution des noms | Installer, créer des zones et enregistrements, configurer les redirecteurs et mises à jour |
| GPO | Gestion centralisée des paramètres Windows | Créer, lier, filtrer, gérer l'héritage et administrer par PowerShell |
| PSO | Stratégie de mot de passe affinée | Créer et appliquer une stratégie à des utilisateurs/groupes concernés |
| Serveur de fichiers | Stockage et partage de fichiers | Configurer les espaces de stockage, disques virtuels et partages |
| FSRM | Gestion des quotas et filtrage de fichiers | Installer le rôle, créer des quotas et configurer le File Screening |
| DFS | Accès distribué aux dossiers | Installer et configurer un espace de noms, comprendre la réplication |
| WDS | Déploiement de Windows par le réseau | Installer/configurer WDS, ajouter les images, capturer et déployer en PXE/multicast |

---

# 2. DHCP — Dynamic Host Configuration Protocol

## 2.1 Définition et rôle

**DHCP** attribue automatiquement aux clients les paramètres réseau nécessaires :
- Adresse IP
- Masque de sous-réseau
- Passerelle par défaut
- Serveurs DNS
- Nom de domaine DNS et autres options configurées

## 2.2 Fonctionnement : DORA

| Étape | Message | Rôle |
|---|---|---|
| 1 | Discover | Le client recherche un serveur DHCP |
| 2 | Offer | Le serveur propose une configuration IP |
| 3 | Request | Le client demande l'adresse proposée |
| 4 | Acknowledge | Le serveur confirme le bail |

**À mémoriser :** Discover → Offer → Request → Acknowledge.

## 2.3 Étendue DHCP (Scope)

Une **étendue** définit la plage d'adresses qu'un serveur DHCP peut distribuer.

| Élément | Explication |
|---|---|
| Plage d'adresses | Première et dernière adresse de la plage distribuable |
| Exclusion | Adresse ou plage que DHCP ne doit pas attribuer |
| Réservation | Adresse réservée à un client identifié, généralement par son adresse MAC |
| Bail (Lease) | Durée pendant laquelle le client peut utiliser l'adresse |
## 2.4 Options DHCP

| Option | Fonction |
|---|---|
| Router | Passerelle par défaut |
| DNS Servers | Serveurs DNS |
| DNS Domain Name | Nom du domaine DNS |
| Lease Duration | Durée du bail |
| NTP Servers | Serveurs de temps, si configurés |
| TFTP Server Name | Nom du serveur TFTP, selon le scénario |

## 2.5 Haute disponibilité

| Mode         | Fonctionnement                                                       |
| ------------ | -------------------------------------------------------------------- |
| Load Balance | Deux serveurs actifs se partagent les demandes                       |
| Hot Standby  | Un serveur principal distribue les adresses ; l'autre est en attente |
| Split Scope  | La plage d'adresses est divisée entre plusieurs serveurs DHCP        |

**Attention :** Split Scope est une méthode historique de répartition. Load Balance et Hot Standby sont les modes de basculement DHCP.
## 2.6 Sauvegarde et restauration

Le programme demande de connaître la sauvegarde et la restauration de la base DHCP.

Chemin habituel de la base :
```text
C:\Windows\System32\dhcp\
```

# 3. DNS — Domain Name System

## 3.1 Définition et rôle

**DNS** traduit les noms de domaine ou de machines en adresses IP. Il permet  aux clients de localiser les services et les machines d'un réseau.

Exemple :
```text
srv1.formation.local → 192.168.1.90
```

## 3.2 Espace de noms et processus de résolution

- **Espace de noms DNS :** organisation hiérarchique des noms.
- **Résolution DNS :** processus utilisé pour retrouver l'information associée à un nom.
- Le serveur peut répondre depuis ses données locales ou interroger/contacter d'autres serveurs selon le type de requête et sa configuration.

## 3.3 Types de zones

| Zone | Rôle |
|---|---|
| Directe (Forward Lookup) | Résout un nom en adresse IP |
| Inverse (Reverse Lookup) | Résout une adresse IP en nom |
| Principale (Primary) | Contient la copie modifiable de la zone |
| Secondaire (Secondary) | Contient une copie en lecture seule de la zone |
| Stub | Contient les informations nécessaires pour localiser les serveurs faisant autorité |

## 3.4 Enregistrements DNS

| Type | Fonction |
|---|---|
| A | Nom vers adresse IPv4 |
| AAAA | Nom vers adresse IPv6 |
| PTR | Adresse IP vers nom |
| CNAME | Alias vers un autre nom |
| MX | Serveur de messagerie |
| NS | Serveur DNS faisant autorité pour la zone |
| SOA | Informations principales de la zone |

Le programme pratique cite explicitement les enregistrements **A, AAAA, PTR et CNAME**.

## 3.5 Requêtes DNS

- **Récursive :** le serveur interrogé doit fournir une réponse finale ou une erreur.
- يجب على الخادم المُستعلَم تقديم إجابة نهائية أو خطأ.
- **Itérative :** le serveur peut renvoyer une référence vers un autre serveur DNS.
يمكن للخادم إرجاع مرجع إلى خادم DNS آخر.
## 3.6 Transfert de zone

Le transfert de zone copie les données DNS vers un autre serveur.
- **AXFR :** transfert complet.
- **IXFR :** transfert des modifications.

## 3.7 Redirecteurs et mises à jour de zone

- **Redirecteur (Forwarder) :** transmet les requêtes qu'un serveur ne peut pas résoudre localement à un autre serveur DNS.
- **Mise à jour dynamique :** permet la mise à jour des enregistrements DNS par les clients ou services autorisés.
- **Nettoyage automatique (Scavenging) :** élimine les enregistrements dynamiques obsolètes selon la configuration.
- **DNS Policy :** permet d'appliquer des règles de réponse DNS selon le scénario configuré.

# here
# 4. GPO et PSO

## 4.1 GPO — définition

Une **GPO (Group Policy Object)** est un ensemble de **paramètres** appliqués automatiquement aux **utilisateurs** et aux **ordinateurs** dans Active Directory.

## 4.2 Ordre d'application : LSDOU

| Lettre | Niveau |
|---|---|
| L | Local |
| S | Site |
| D | Domaine |
| OU | Organizational Unit |

**À mémoriser :** Local → Site → Domaine → OU.

En règle générale, les paramètres appliqués plus tard peuvent remplacer les paramètres précédents en cas de conflit, sous réserve des options de priorité et d'héritage.

## 4.3 Concepts de gestion

| Concept | Fonction |
|---|---|
| GPMC | Console graphique de gestion des GPO (`gpmc.msc`) |
| Starter GPO | Modèle de paramètres servant de point de départ à une nouvelle GPO |
| Group Policy Preferences | Permet de configurer certains paramètres/préférences utilisateur ou ordinateur |
| Security Filtering | Limite les utilisateurs/ordinateurs autorisés à appliquer la GPO |
| WMI Filter | Conditionne l'application selon des critères système WMI |
| Block Inheritance | Bloque l'héritage des GPO des niveaux supérieurs, sauf liens forcés |
| Enforced | Force la priorité d'un lien et empêche qu'un blocage d'héritage ordinaire le neutralise |
| Link Order | Définit la priorité des liens GPO au même niveau ; le lien de rang 1 a la priorité la plus élevée |

## 4.4 PSO — Password Settings Object

Une **PSO** est une stratégie de mot de passe affinée. Elle permet de définir des paramètres de mot de passe et de verrouillage différents pour des utilisateurs ou groupes de sécurité globaux ciblés.

**À retenir :** la valeur numérique de précédence la plus faible est prioritaire lorsqu'il faut départager plusieurs PSO applicables.

## 4.5 Commandes de diagnostic

```powershell
# Actualiser les stratégies
gpupdate /force

# Afficher un résumé des stratégies appliquées
gpresult /r

# Générer un rapport HTML
gpresult /h C:\Temp\rapport-gpo.html

# Ouvrir le jeu de stratégies résultant
rsop.msc

# Ouvrir la console GPMC
gpmc.msc
```

# 5. Serveur de fichiers et FSRM

## 5.1 Serveur de fichiers — notions du programme

Le PDF inclut des notions de serveur de fichiers qui doivent être révisées en plus de FSRM :

- Fonctionnalité **Storage Spaces** (espaces de stockage).
- Options de configuration des disques virtuels via l'interface graphique.
- Création d'un répertoire partagé.
- Affichage des partages selon les droits d'accès.
- Déduplication des données.
- Installation du rôle de serveur de fichiers.

### Partage et permissions

Un dossier partagé est accessible sur le réseau via un chemin UNC, par exemple :

```text
\\SRV-FICHIERS\Commun
```

Il faut distinguer :
- **Permissions de partage :** contrôlent l'accès via le partage réseau.
- **Permissions NTFS :** contrôlent les droits sur le système de fichiers.
- Les droits effectifs résultent de la combinaison des autorisations applicables ; il faut vérifier les deux niveaux.

### Storage Spaces et disques virtuels

**Storage Spaces** permet de regrouper des disques physiques dans un pool de stockage, puis de créer des disques virtuels à partir de ce pool.

Le programme demande notamment de savoir configurer les disques virtuels graphiquement. Les détails dépendent du scénario et des disques disponibles dans le laboratoire.

### Déduplication

La **déduplication des données** réduit l'espace occupé en évitant de stocker plusieurs fois des blocs de données identiques, selon la configuration du volume.

> **Distinction importante :** la déduplication est une fonctionnalité du serveur de fichiers ; ce n'est pas une fonction de quota ou de File Screening FSRM.

## 5.2 FSRM — définition

**FSRM (File Server Resource Manager)** est un rôle Windows Server qui permet de gérer et contrôler l'utilisation du stockage sur un serveur de fichiers.

Fonctions principales :
- Quotas.
- File Screening (filtrage de fichiers).
- Groupes de fichiers.
- Classification et rapports de stockage.
- Tâches de gestion de fichiers.

## 5.3 Quotas

Un quota limite l'espace utilisé par un dossier ou un volume.

| Type | Fonction |
|---|---|
| Hard Quota | Bloque l'écriture lorsque la limite est atteinte |
| Soft Quota | Sert au suivi/aux alertes sans bloquer nécessairement l'écriture |

## 5.4 File Screening

Le File Screening contrôle les types de fichiers autorisés dans un dossier.

| Mode | Fonction |
|---|---|
| Active Screening | Bloque les fichiers correspondant aux règles |
| Passive Screening | Surveille et journalise sans bloquer |

Exemple : empêcher le stockage de certains fichiers vidéo dans un dossier partagé.

## 5.5 Installation graphique de FSRM

1. Ouvrir **Server Manager**.
2. Choisir **Add Roles and Features**.
3. Ouvrir **File and Storage Services**.
4. Ouvrir **File and iSCSI Services**.
5. Sélectionner **File Server Resource Manager**.
6. Installer le rôle.

# 6. DFS — Distributed File System

## 6.1 Définition et intérêt

**DFS** permet de présenter des dossiers partagés situés sur plusieurs serveurs à travers un espace de noms commun. Il simplifie l'accès aux ressources dans une infrastructure distribuée.

## 6.2 Deux composants

| Composant | Fonction |
|---|---|
| DFS Namespace (DFS-N) | Présente les dossiers partagés sous un chemin logique commun |
| DFS Replication (DFS-R) | Réplique les dossiers entre plusieurs serveurs |

**À mémoriser :** DFS-N organise l'accès ; DFS-R réplique les données.

## 6.3 Types d'espaces de noms

| Type | Caractéristique |
|---|---|
| Standalone Namespace | Espace de noms hébergé par un serveur |
| Domain-based Namespace | Espace de noms intégré à Active Directory et au domaine |

## 6.4 Exemple

Accès directs aux partages :
```text
\\SRV-FICHIERS-01\Comptabilite
\\SRV-FICHIERS-02\RH
```

Chemin logique commun :
```text
\\formation.local\Partages\Comptabilite
\\formation.local\Partages\RH
```

## 6.5 Installation et configuration

1. Installer **DFS Namespaces** et/ou **DFS Replication** selon le besoin.
2. Créer l'espace de noms.
3. Configurer l'espace de noms.
4. Ajouter les dossiers et leurs cibles.
5. Pour la réplication, configurer les membres, connexions et dossiers répliqués.

# here
# 7. WDS — Windows Deployment Services

## 7.1 Définition et rôle

**WDS** permet de déployer Windows sur des ordinateurs clients à travers le réseau, notamment grâce au démarrage PXE.

## 7.2 PXE — Preboot Execution Environment

PXE permet à un ordinateur de démarrer depuis le réseau.

Processus général :
1. Le client démarre en PXE.
2. Il obtient les paramètres réseau nécessaires.
3. Il contacte le service de déploiement.
4. Il charge une image de démarrage.
5. Il sélectionne une image d'installation.
6. L'installation de Windows commence.

## 7.3 Types d'images

| Image | Fonction |
|---|---|
| Boot Image | Démarre l'environnement d'installation |
| Install Image | Contient l'image Windows à installer |
| Capture Image | Permet de capturer l'image d'un ordinateur de référence |

## 7.4 Ordinateur de référence et capture

Un **ordinateur de référence** est une machine préparée avec Windows et les configurations souhaitées. Une image de capture permet de capturer son installation pour la déployer sur d'autres ordinateurs.

## 7.5 Multicast

Le multicast permet de distribuer une image à plusieurs clients simultanément, afin d'éviter de transmettre séparément les mêmes données à chaque client lorsque le scénario et le réseau le permettent.

## 7.6 Fichier `unattend.xml`

Le fichier de réponse `unattend.xml` automatise certaines étapes de l'installation de Windows, par exemple :
- Paramètres régionaux.
- Nom de l'ordinateur.
- Paramètres d'installation.
- Réponses à certaines étapes de configuration.

## 7.7 Installation et configuration

1. Installer le rôle WDS.
2. Configurer le serveur WDS.
3. Ajouter une image de démarrage.
4. Ajouter une image d'installation.
5. Créer une image de capture.
6. Capturer un poste de référence.
7. Configurer et tester le démarrage PXE.
8. Étudier le déploiement multicast et le fichier `unattend.xml`.

## 7.8 💻 PowerShell / WDSUtil — commandes de consultation et d'exemple

```powershell
# Installer WDS
Install-WindowsFeature -Name WDS -IncludeManagementTools
```

Exemples WDSUtil (adapter les chemins aux fichiers réellement disponibles) :

```text
# Initialiser WDS avec le dossier de déploiement
wdsutil /Initialize-Server /RemInst:"D:\RemoteInstall"

# Afficher la configuration du serveur
wdsutil /Get-Server /Show:Config

# Ajouter une image de démarrage
wdsutil /Add-Image /ImageFile:"D:\Images\boot.wim" /ImageType:Boot

# Ajouter une image d'installation
wdsutil /Add-Image /ImageFile:"D:\Images\install.wim" /ImageType:Install /ImageGroup:"Windows"

# Lister les images de démarrage
wdsutil /Get-AllImages /ImageType:Boot

# Lister les images d'installation
wdsutil /Get-AllImages /ImageType:Install
```

> Les chemins et options doivent correspondre au média Windows et à la configuration WDS du laboratoire. Le PDF demande aussi la création d'une image de capture, la capture d'un poste de référence et le déploiement multicast : ces étapes ne sont pas remplacées par les seules commandes d'ajout d'images ci-dessus.

---

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

---

# 8. Méthode pour les travaux pratiques

Pour chaque exercice Windows Server :

```text
1. Lire précisément la consigne.
2. Identifier le rôle ou service concerné.
3. Vérifier si le rôle est déjà installé.
4. Installer le rôle si nécessaire.
5. Créer l'objet demandé (étendue, zone, GPO, quota, namespace, image...).
6. Configurer ses paramètres.
7. Activer ou lier l'objet si nécessaire.
8. Vérifier avec une commande Get-* ou un outil graphique.
9. Tester depuis un client lorsque c'est demandé.
10. Noter le résultat et corriger les erreurs.
```

## Commandes de vérification rapides

| Technologie | Vérification |
|---|---|
| DHCP | `Get-DhcpServerv4Scope` |
| DNS | `Get-DnsServerZone` |
| GPO | `Get-GPO -All` et `gpresult /r` |
| PSO | `Get-ADFineGrainedPasswordPolicy -Filter *` |
| FSRM | `Get-FsrmQuota` |
| DFS | `Get-DfsnRoot` |
| WDS | `wdsutil /Get-Server /Show:Config` |

---

1. Quel est le rôle du DHCP ?
2. Que signifie DORA ?
3. Quelle est la différence entre une exclusion et une réservation DHCP ?
4. Quelle est la différence entre Load Balance et Hot Standby ?
5. Pourquoi sauvegarder la base DHCP ?
6. Quel est le rôle du DNS ?
7. Quelle est la différence entre une zone directe et une zone inverse ?
8. Quelle est la différence entre une zone principale et une zone secondaire ?
9. À quoi servent les enregistrements A, AAAA, PTR et CNAME ?
10. Quelle est la différence entre AXFR et IXFR ?
11. À quoi sert un redirecteur DNS ?
12. À quoi servent les mises à jour dynamiques et le nettoyage DNS ?
13. Que signifie LSDOU ?
14. Quel est le rôle de GPMC ?
15. Quelle est la différence entre Enforced et Block Inheritance ?
16. À quoi servent les filtres de sécurité et les filtres WMI ?
17. Qu'est-ce qu'une PSO ?
18. Quelle est la différence entre permissions de partage et permissions NTFS ?
19. À quoi sert Storage Spaces ?
20. Quel est l'objectif de la déduplication ?
21. Quelle est la différence entre Hard Quota et Soft Quota ?
22. Quelle est la différence entre Active Screening et Passive Screening ?
23. Quelle est la différence entre DFS-N et DFS-R ?
24. Quelle est la différence entre un espace de noms Standalone et Domain-based ?
25. Quel est le rôle de WDS ?
26. Quelle est la différence entre Boot Image et Install Image ?
27. À quoi sert une Capture Image ?
28. Quel est le rôle de PXE ?
29. Quel est l'intérêt du multicast ?
30. À quoi sert le fichier `unattend.xml` ?

---

# 10. Checklist avant l'examen

## DHCP
- [ ] Expliquer DORA.
- [ ] Créer une étendue et configurer ses options.
- [ ] Configurer une exclusion et une réservation.
- [ ] Expliquer les modes de haute disponibilité.
- [ ] Sauvegarder et restaurer la base DHCP.
- [ ] Effectuer les opérations principales en PowerShell.

## DNS
- [ ] Expliquer l'espace de noms et le processus de résolution.
- [ ] Créer des zones directes et inverses.
- [ ] Différencier zones principales et secondaires.
- [ ] Créer les enregistrements A, AAAA, PTR et CNAME.
- [ ] Expliquer les requêtes et transferts de zones.
- [ ] Configurer les redirecteurs et mises à jour de zone.
- [ ] Activer/tester le nettoyage automatique et créer une politique DNS.
- [ ] Effectuer les opérations principales en PowerShell.

## GPO / PSO
- [ ] Expliquer LSDOU et l'ordre d'application.
- [ ] Utiliser GPMC et comprendre Starter GPO / Preferences.
- [ ] Créer et lier une GPO à une OU.
- [ ] Supprimer une liaison sans confondre avec la suppression de la GPO.
- [ ] Utiliser les filtres et le blocage d'héritage.
- [ ] Créer et appliquer une PSO.
- [ ] Créer, lier, modifier et supprimer une GPO avec PowerShell.

## Serveur de fichiers / FSRM
- [ ] Expliquer Storage Spaces et les disques virtuels.
- [ ] Créer un dossier partagé.
- [ ] Distinguer permissions de partage et NTFS.
- [ ] Expliquer la déduplication.
- [ ] Installer le rôle serveur de fichiers / FSRM.
- [ ] Créer un quota.
- [ ] Configurer un filtrage de fichiers.

## DFS
- [ ] Expliquer le rôle de DFS.
- [ ] Différencier DFS-N et DFS-R.
- [ ] Identifier Standalone et Domain-based.
- [ ] Installer et configurer un espace de noms.
- [ ] Expliquer les scénarios de réplication.

## WDS
- [ ] Installer et configurer WDS.
- [ ] Ajouter une image de démarrage et une image d'installation.
- [ ] Créer une image de capture.
- [ ] Capturer un poste de référence.
- [ ] Expliquer PXE et multicast.
- [ ] Expliquer la structure et le rôle de `unattend.xml`.

---

