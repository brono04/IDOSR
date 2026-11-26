# M202 — Préparation pratique EFM
## Windows Server : DHCP, DNS, GPO/PSO, Serveur de fichiers/FSRM, DFS et WDS

> **Objectif :** s'entraîner à réaliser les opérations pratiques demandées dans le référentiel M202, puis vérifier le résultat avec des commandes et des tests client.
>
> **Base officielle :** PDF OFPPT *supo du core M202.pdf*, pages 8 à 14 (compétences D à I).

> **Important :** les étapes marquées **Complément pratique** détaillent une manière courante de réaliser les objectifs. Les paramètres exacts peuvent varier selon la version de Windows Server et le laboratoire du formateur. Ne pas exécuter les commandes de production sans validation.

---
## 0. Préparer le laboratoire

### 0.1 Topologie proposée

| Machine | Rôle | Adresse IPv4 proposée |
|---|---|---:|
| SRV-DC | AD DS, DNS, DHCP, GPO | `192.168.10.10/24` |
| SRV-FS | Serveur de fichiers, FSRM, DFS | `192.168.10.20/24` |
| SRV-WDS | WDS (peut être SRV-DC dans un petit lab) | `192.168.10.30/24` |
| Client-01 | Windows 10/11 client | DHCP |

- Domaine d'exemple : `entreprise.local`
- Réseau : `192.168.10.0/24`
- Passerelle : `192.168.10.1` (si un routeur existe)
- DNS du domaine : `192.168.10.10`
- Plage DHCP : `192.168.10.100` à `192.168.10.200`

**Règles de laboratoire**
1. Utiliser des machines virtuelles et des snapshots.
2. Attribuer des IP statiques aux serveurs.
3. Vérifier que les noms, IP et DNS sont cohérents.
4. Exécuter PowerShell en administrateur.
5. Remplacer les noms, IP et chemins d'exemple par ceux du laboratoire.
6. Ne pas activer DHCP sur un réseau partagé ou réel sans autorisation : un second serveur DHCP peut perturber le réseau.

### 0.2 Vérifications initiales

```powershell
hostname
ipconfig /all
Get-NetIPConfiguration
Get-Service DNS,DHCPServer -ErrorAction SilentlyContinue
```

Depuis le client :

```powershell
ipconfig /all
ping 192.168.10.10
nslookup entreprise.local
```

---

# 1. DHCP — Installer, configurer, réserver, sauvegarder et restaurer

## 1.1 Installer le rôle DHCP

**Interface graphique**
1. Ouvrir **Server Manager**.
2. `Manage` → `Add Roles and Features`.
3. Choisir `Role-based or feature-based installation`.
4. Sélectionner le serveur.
5. Cocher **DHCP Server** et accepter les fonctionnalités proposées.
6. Installer.
7. Cliquer sur **Complete DHCP configuration** si l'assistant le demande.
8. Dans la console DHCP, autoriser le serveur dans Active Directory si nécessaire.

**PowerShell**

```powershell
Install-WindowsFeature DHCP -IncludeManagementTools
Get-WindowsFeature DHCP
```

Pour un domaine AD, l'autorisation doit être réalisée avec un compte autorisé :

```powershell
Add-DhcpServerInDC -DnsName "SRV-DC.entreprise.local" -IpAddress 192.168.10.10
Get-DhcpServerInDC
```

## 1.2 Créer une étendue IPv4

**Paramètres de l'exercice**

- Nom : `LAN-Entreprise`
- Début : `192.168.10.100`
- Fin : `192.168.10.200`
- Masque : `255.255.255.0`
- Exclusion : `192.168.10.100` à `192.168.10.109`
- Passerelle (option 003) : `192.168.10.1`
- DNS (option 006) : `192.168.10.10`
- Nom de domaine (option 015) : `entreprise.local`

**Interface graphique**
1. Ouvrir `Tools` → `DHCP`.
2. Développer le serveur → `IPv4`.
3. Clic droit `IPv4` → `New Scope`.
4. Saisir le nom et la description.
5. Saisir l'adresse de début, de fin et le masque.
6. Ajouter la plage d'exclusion.
7. Définir la durée du bail selon l'énoncé.
8. Configurer les options 003, 006 et 015.
9. Activer l'étendue.

**PowerShell**

```powershell
Add-DhcpServerv4Scope `
  -Name "LAN-Entreprise" `
  -StartRange 192.168.10.100 `
  -EndRange 192.168.10.200 `
  -SubnetMask 255.255.255.0 `
  -State Active

Add-DhcpServerv4ExclusionRange `
  -ScopeId 192.168.10.0 `
  -StartRange 192.168.10.100 `
  -EndRange 192.168.10.109

Set-DhcpServerv4OptionValue -ScopeId 192.168.10.0 `
  -Router 192.168.10.1 `
  -DnsServer 192.168.10.10 `
  -DnsDomain "entreprise.local"

Get-DhcpServerv4Scope
Get-DhcpServerv4OptionValue -ScopeId 192.168.10.0
```

> Si le réseau ou la plage diffère, `ScopeId` doit être l'adresse réseau de l'étendue, pas l'adresse du serveur.

## 1.3 Réserver un bail DHCP

Une réservation associe une adresse IP à l'identifiant du client (souvent l'adresse MAC).

1. Sur le client, récupérer l'adresse physique :

```powershell
ipconfig /all
```

2. Dans DHCP : `IPv4` → étendue → `Reservations` → `New Reservation`.
3. Saisir le nom, l'adresse IP et l'adresse MAC/Client ID.
4. Choisir le type pris en charge par le laboratoire, puis valider.

**PowerShell — exemple**

```powershell
Add-DhcpServerv4Reservation `
  -ScopeId 192.168.10.0 `
  -IPAddress 192.168.10.150 `
  -ClientId "00-0C-29-8E-B5-82" `
  -Description "Imprimante-01"

Get-DhcpServerv4Reservation -ScopeId 192.168.10.0
```

L'adresse réservée doit être cohérente avec la conception de l'étendue et ne doit pas être utilisée par un autre équipement.

## 1.4 Tester depuis le client

```cmd
ipconfig /release
ipconfig /renew
ipconfig /all
```

Vérifier :
- adresse dans la plage prévue ;
- masque correct ;
- passerelle correcte ;
- DNS correct ;
- suffixe DNS correct.

Sur le serveur :

```powershell
Get-DhcpServerv4Lease -ScopeId 192.168.10.0
```

## 1.5 Sauvegarder et restaurer la base DHCP

**Console graphique**
- DHCP → clic droit sur le serveur → `Backup` → choisir un dossier.
- Pour restaurer : clic droit sur le serveur → `Restore` → choisir le dossier de sauvegarde. Respecter les consignes de l'assistant et du laboratoire.

**PowerShell**

```powershell
Backup-DhcpServer -ComputerName "SRV-DC" -Path "C:\DHCP-Backup"
```

Pour une restauration, utiliser le dossier de sauvegarde approprié et suivre la procédure prévue pour le serveur concerné :

```powershell
Restore-DhcpServer -ComputerName "SRV-DC" -Path "C:\DHCP-Backup" -Force
```

> Effectuer la restauration uniquement dans un lab ou pendant une fenêtre autorisée. Vérifier ensuite les étendues, options, réservations et baux.

## 1.6 Haute disponibilité DHCP — savoir expliquer

Le référentiel demande les **notions de haute disponibilité**. Savoir expliquer :
- pourquoi un DHCP unique constitue un point de défaillance ;
- le principe du basculement DHCP (failover) entre deux serveurs ;
- les modes courants : équilibrage de charge et secours (hot standby) ;
- la nécessité de tester la réplication et la continuité de service.

**À retenir :** deux serveurs DHCP ne doivent pas distribuer indépendamment la même plage sans configuration coordonnée.

### Validation DHCP

- [ ] Rôle DHCP installé.
- [ ] Serveur autorisé dans AD si nécessaire.
- [ ] Étendue créée et activée.
- [ ] Exclusion configurée.
- [ ] Options passerelle/DNS/domaine configurées.
- [ ] Réservation créée (exemple : `192.168.10.150`, adresse comprise dans l'étendue).
- [ ] Client obtient une adresse correcte.
- [ ] Sauvegarde réalisée et procédure de restauration comprise.

---

# 2. DNS — Zones, enregistrements, transfert, nettoyage et politique DNS

## 2.1 Installer le rôle DNS

**GUI :** Server Manager → Add Roles and Features → **DNS Server** → Install.

```powershell
Install-WindowsFeature DNS -IncludeManagementTools
Get-WindowsFeature DNS
```

## 2.2 Créer une zone directe principale

**GUI**
1. Ouvrir `Tools` → `DNS`.
2. Développer le serveur.
3. Clic droit `Forward Lookup Zones` → `New Zone`.
4. Choisir `Primary Zone`.
5. Si le serveur est contrôleur de domaine, choisir l'intégration AD si demandée et adaptée au lab.
6. Nom : `entreprise.local`.
7. Choisir le type de mises à jour dynamiques demandé par l'énoncé.

**PowerShell**

```powershell
Add-DnsServerPrimaryZone -Name "entreprise.local" -ZoneFile "entreprise.local.dns"
Get-DnsServerZone
```

Pour une zone intégrée à Active Directory, sur un serveur DNS/AD DS :

```powershell
Add-DnsServerPrimaryZone -Name "entreprise.local" -ReplicationScope "Domain"
```

Ne pas créer deux fois la même zone. Choisir une seule méthode selon l'environnement.

## 2.3 Créer une zone inversée

Pour le réseau `192.168.10.0/24`, le nom de zone inversée est `10.168.192.in-addr.arpa`.

**GUI :** `Reverse Lookup Zones` → `New Zone` → Primary → IPv4 Reverse Lookup Zone → Network ID `192.168.10`.

```powershell
Add-DnsServerPrimaryZone -NetworkId "192.168.10.0/24" -ZoneFile "10.168.192.in-addr.arpa.dns"
```

## 2.4 Créer les enregistrements DNS A, AAAA, CNAME et PTR

Exemples :
- A : `srv-fs.entreprise.local` → `192.168.10.20`
- CNAME : `fichiers.entreprise.local` → `srv-fs.entreprise.local`
- PTR : `192.168.10.20` → `srv-fs.entreprise.local`
- AAAA : associe un nom à une adresse IPv6 (utiliser une IPv6 réelle du lab).

```powershell
Add-DnsServerResourceRecordA -ZoneName "entreprise.local" `
  -Name "srv-fs" -IPv4Address "192.168.10.20"

Add-DnsServerResourceRecordCName -ZoneName "entreprise.local" `
  -Name "fichiers" -HostNameAlias "srv-fs.entreprise.local"

Add-DnsServerResourceRecordPtr -ZoneName "10.168.192.in-addr.arpa" `
  -Name "20" -PtrDomainName "srv-fs.entreprise.local"
```

**AAAA — modèle à adapter avec l'IPv6 réelle :**

```powershell
Add-DnsServerResourceRecordAAAA -ZoneName "entreprise.local" `
  -Name "srv-ipv6" -IPv6Address "2001:db8:10::20"
```

`2001:db8::/32` est un préfixe de documentation : ne pas l'utiliser comme adresse IPv6 de production.

Vérifier :

```powershell
Get-DnsServerResourceRecord -ZoneName "entreprise.local"
Resolve-DnsName "srv-fs.entreprise.local"
Resolve-DnsName "fichiers.entreprise.local"
nslookup 192.168.10.20
```

## 2.5 Zone secondaire et transfert de zone

**Objectif :** comprendre la différence entre zone principale et secondaire, puis créer une zone secondaire sur un second serveur DNS.

Prérequis :
- deux serveurs DNS ;
- zone principale sur DNS1 ;
- transfert de zone autorisé sur DNS1 vers l'adresse de DNS2 ;
- connectivité réseau et pare-feu adaptés.

**Sur DNS1 (zone principale)**
1. DNS Manager → zone `entreprise.local` → Properties.
2. Onglet `Zone Transfers`.
3. Autoriser les transferts uniquement vers les serveurs spécifiés.
4. Ajouter l'adresse IP de DNS2.

**Sur DNS2**
1. `Forward Lookup Zones` → `New Zone`.
2. Choisir `Secondary Zone`.
3. Nom : `entreprise.local`.
4. Master DNS server : adresse de DNS1.
5. Terminer puis vérifier le chargement de la zone.

**Contrôles**
```powershell
Get-DnsServerZone
Get-DnsServerResourceRecord -ZoneName "entreprise.local"
```

> Une zone secondaire est une copie en lecture seule obtenue par transfert de zone. Ne pas confondre avec une zone principale intégrée à AD.

## 2.6 Redirecteurs DNS

Dans DNS Manager : clic droit serveur → Properties → `Forwarders` → ajouter le DNS amont autorisé par le lab.

```powershell
Set-DnsServerForwarder -IPAddress 1.1.1.1,8.8.8.8
Get-DnsServerForwarder
```

> Utiliser uniquement les adresses autorisées dans le laboratoire. Un redirecteur transmet certaines requêtes à un autre serveur DNS.

## 2.7 Mises à jour dynamiques et nettoyage automatique (scavenging)

Le nettoyage aide à supprimer les enregistrements dynamiques obsolètes, après configuration correcte des intervalles.

**GUI**
1. DNS Manager → clic droit serveur → `Set Aging/Scavenging for All Zones` (selon version).
2. Activer le scavenging au niveau serveur et définir un intervalle adapté.
3. Sur la zone : Properties → `Aging` → activer le nettoyage.
4. Vérifier les timestamps des enregistrements dynamiques.
5. Tester uniquement dans le lab et ne pas réduire excessivement les intervalles.

**PowerShell — inspection et activation à adapter**

```powershell
Get-DnsServerScavenging
Get-DnsServerZoneAging -Name "entreprise.local"
```

Exemple de configuration (valeurs à adapter au lab) :

```powershell
Set-DnsServerScavenging -ScavengingState $true -ScavengingInterval 7.00:00:00
Set-DnsServerZoneAging -Name "entreprise.local" -AgingState $true `
  -NoRefreshInterval 7.00:00:00 -RefreshInterval 7.00:00:00
```

## 2.8 Politique DNS

Le référentiel demande de créer une politique DNS. Les politiques DNS peuvent appliquer des réponses différentes selon des critères (par exemple, sous-réseau client). Le scénario exact dépend du besoin et de la version du serveur.

**Préparation pratique**
1. Identifier le besoin : séparer les réponses selon le sous-réseau client, par exemple.
2. Créer un client subnet DNS.
3. Créer une zone scope et les enregistrements nécessaires.
4. Créer la politique qui associe le sous-réseau à la zone scope.
5. Tester avec un client de chaque sous-réseau.

**Exemple PowerShell complet — réponse DNS différente selon le sous-réseau**

Dans cet exemple, le serveur répond à `intranet.entreprise.local` avec l'adresse normale `192.168.10.20`, mais les clients du sous-réseau `192.168.10.0/24` reçoivent l'adresse alternative `192.168.10.30`.

```powershell
# Déclarer le sous-réseau client
Add-DnsServerClientSubnet -Name "LAN-Lab" -IPv4Subnet "192.168.10.0/24"

# Créer une zone scope pour les réponses alternatives
Add-DnsServerZoneScope -ZoneName "entreprise.local" -Name "LAN-Scope"

# Ajouter l'enregistrement dans la zone scope
Add-DnsServerResourceRecord -ZoneName "entreprise.local" `
  -ZoneScope "LAN-Scope" -A -Name "intranet" -IPv4Address "192.168.10.30"

# Associer le sous-réseau à la zone scope
Add-DnsServerQueryResolutionPolicy -Name "Policy-LAN-Lab" `
  -ZoneName "entreprise.local" -Action ALLOW `
  -ClientSubnet "EQ,LAN-Lab" -ZoneScope "LAN-Scope,1"

# Vérifier la configuration
Get-DnsServerClientSubnet
Get-DnsServerZoneScope -ZoneName "entreprise.local"
Get-DnsServerQueryResolutionPolicy -ZoneName "entreprise.local"
```

Créer au préalable l'enregistrement A normal `intranet` dans la zone par défaut, et vérifier que `192.168.10.30` correspond à un hôte de test du laboratoire. Tester la résolution depuis un client du sous-réseau ciblé et depuis un autre sous-réseau.

**Cmdlets utiles**

```powershell
Get-Command *DnsServer*Policy*
Get-Help Add-DnsServerQueryResolutionPolicy -Examples
```

> Cet exemple illustre une politique au niveau d'une zone. Les paramètres et la disponibilité des politiques DNS dépendent de la version Windows Server. Vérifier les exemples d'aide locale avant exécution.

### Validation DNS

- [ ] Rôle DNS installé.
- [ ] Zone directe principale créée.
- [ ] Zone inversée créée.
- [ ] Zone secondaire créée sur DNS2 et transfert fonctionnel.
- [ ] Enregistrements A, AAAA, PTR, CNAME créés.
- [ ] Résolution directe et inverse testée.
- [ ] Redirecteur configuré.
- [ ] Mises à jour dynamiques et scavenging compris/testés.
- [ ] Politique DNS préparée et testée selon le scénario.
- [ ] Opérations principales réalisées en PowerShell.

---

# 3. GPO et PSO — Créer, lier, filtrer, bloquer l'héritage et administrer

## 3.1 Comprendre l'ordre d'application

Ordre classique **LSDOU** :
1. Local
2. Site
3. Domaine
4. OU (de l'OU parente vers l'OU enfant)

Une stratégie appliquée plus tard peut prendre le dessus en cas de conflit, selon les liens, l'héritage, le filtrage et l'option Enforced.

## 3.2 Créer une OU et des utilisateurs de test

```powershell
Import-Module ActiveDirectory
New-ADOrganizationalUnit -Name "OU-Lab" -Path "DC=entreprise,DC=local"
New-ADUser -Name "Test-GPO" -SamAccountName "Test-GPO" `
  -Path "OU=OU-Lab,DC=entreprise,DC=local" `
  -Enabled $true -AccountPassword (Read-Host -AsSecureString "Mot de passe")
```

Adapter le DN au domaine réel. Créer un utilisateur de test non privilégié.

## 3.3 Créer et lier un GPO

**GUI**
1. Ouvrir `Group Policy Management` (`gpmc.msc`).
2. Développer Forest → Domains → domaine.
3. Clic droit sur `Group Policy Objects` → `New`.
4. Nom : `GPO-LAB-Restriction`.
5. Clic droit sur l'OU cible → `Link an Existing GPO`.
6. Sélectionner le GPO.

**PowerShell**

```powershell
Import-Module GroupPolicy
New-GPO -Name "GPO-LAB-Restriction"
New-GPLink -Name "GPO-LAB-Restriction" `
  -Target "OU=OU-Lab,DC=entreprise,DC=local" -LinkEnabled Yes
Get-GPO -All
Get-GPInheritance -Target "OU=OU-Lab,DC=entreprise,DC=local"
```

## 3.4 Modifier un paramètre GPO

Dans GPMC :
1. Clic droit GPO → `Edit`.
2. Naviguer vers le paramètre demandé par l'énoncé.
3. Ouvrir le paramètre.
4. Choisir `Enabled`, `Disabled` ou `Not Configured` selon le besoin.
5. Fermer l'éditeur.

Exemple de paramètre de test : définir une stratégie de mot de passe/écran de verrouillage ou une restriction non destructive. Le chemin exact dépend du paramètre demandé.

Sur le client :

```cmd
gpupdate /force
gpresult /r
gpresult /h C:\Temp\gpresult.html
```

Dans PowerShell :

```powershell
Get-GPOReport -Name "GPO-LAB-Restriction" -ReportType Html -Path "C:\Temp\GPO-LAB.html"
```

## 3.5 Filtrage de sécurité

1. Dans GPMC, sélectionner le GPO.
2. Dans `Scope` → `Security Filtering`, retirer `Authenticated Users` uniquement si le scénario le demande.
3. Ajouter le groupe ou utilisateur autorisé.
4. Vérifier les autorisations `Read` et `Apply Group Policy`.
5. Tester avec un compte ciblé et un compte non ciblé.

**Attention :** un filtrage mal configuré peut empêcher le GPO de s'appliquer. Ne pas supprimer des permissions sans comprendre le résultat.

## 3.6 Filtrage WMI

1. Créer une requête WMI dans `WMI Filters`.
2. Donner un nom et une description.
3. Ajouter une requête adaptée au critère du scénario (OS, version, matériel).
4. Associer le filtre au GPO dans la liste `WMI Filtering`.
5. Tester sur des machines correspondant et ne correspondant pas au filtre.

**Exemple de requête WMI pour Windows 10 (à adapter) :**

```sql
SELECT * FROM Win32_OperatingSystem WHERE ProductType = "1" AND Caption LIKE "%Windows 10%"
```

## 3.7 Blocage de l'héritage et Enforced

- `Block Inheritance` se configure sur l'OU : empêche l'héritage des GPO venant des conteneurs parents, sauf liens Enforced.
- `Enforced` se configure sur un lien GPO : le lien ne peut pas être bloqué par Block Inheritance et les paramètres du GPO ont une priorité particulière.
- Vérifier `Group Policy Inheritance` dans GPMC pour visualiser le résultat.

**Exercice :** créer un GPO au niveau du domaine, créer un GPO sur une OU enfant, activer Block Inheritance puis comparer le résultat avant/après. Tester ensuite un lien Enforced dans un environnement isolé.

## 3.8 Supprimer un lien ou un GPO

Supprimer uniquement le lien :

```powershell
Remove-GPLink -Name "GPO-LAB-Restriction" `
  -Target "OU=OU-Lab,DC=entreprise,DC=local" -Confirm:$false
```

Supprimer l'objet GPO lui-même :

```powershell
Remove-GPO -Name "GPO-LAB-Restriction" -Confirm:$false
```

Différence à retenir : supprimer un lien ne supprime pas nécessairement l'objet GPO.

## 3.9 Créer un magasin central (Central Store)

**But :** centraliser les fichiers ADMX/ADML utilisés par les éditeurs de stratégie du domaine.

1. Sur un contrôleur de domaine, ouvrir `\\entreprise.local\SYSVOL\entreprise.local\Policies`.
2. Créer le dossier `PolicyDefinitions` s'il n'existe pas.
3. Copier les fichiers ADMX depuis `C:\Windows\PolicyDefinitions` d'un poste/serveur de référence.
4. Copier les dossiers de langue ADML correspondants (par exemple `fr-FR` ou `en-US`).
5. Ouvrir GPMC et vérifier que les modèles administratifs sont disponibles.

Ne pas mélanger des versions incompatibles de modèles ADMX/ADML.

## 3.10 PSO — stratégie de mot de passe affinée

Le PSO (Password Settings Object) permet d'appliquer une politique de mot de passe à des utilisateurs ou groupes spécifiques, plutôt qu'à tout le domaine.

**Exemple PowerShell**

```powershell
Import-Module ActiveDirectory

New-ADFineGrainedPasswordPolicy `
  -Name "PSO-Admins-Lab" `
  -Precedence 10 `
  -MinPasswordLength 12 `
  -ComplexityEnabled $true `
  -PasswordHistoryCount 10 `
  -MaxPasswordAge (New-TimeSpan -Days 60) `
  -MinPasswordAge (New-TimeSpan -Days 1) `
  -LockoutThreshold 5 `
  -LockoutDuration (New-TimeSpan -Minutes 15) `
  -LockoutObservationWindow (New-TimeSpan -Minutes 15)

Add-ADFineGrainedPasswordPolicySubject `
  -Identity "PSO-Admins-Lab" -Subjects "GG-Admins-Lab"

Get-ADFineGrainedPasswordPolicy -Filter *
Get-ADUserResultantPasswordPolicy -Identity "Test-GPO"
```

Créer le groupe de sécurité avant de l'utiliser, et adapter les paramètres à l'énoncé. Vérifier la stratégie résultante sur un utilisateur réellement ciblé.

### Validation GPO/PSO

- [ ] GPO créé.
- [ ] GPO lié à la bonne OU.
- [ ] Paramètre modifié dans l'éditeur.
- [ ] `gpupdate` exécuté et résultat contrôlé par `gpresult`.
- [ ] Filtrage de sécurité testé.
- [ ] Filtre WMI testé.
- [ ] Block Inheritance et Enforced différenciés.
- [ ] Lien supprimé sans confusion avec la suppression du GPO.
- [ ] Central Store créé.
- [ ] PSO créée, affectée et vérifiée.

---

# 4. Serveur de fichiers et FSRM — Partages, quotas, filtrage, déduplication et Storage Spaces

## 4.1 Installer le rôle File Server et FSRM

**GUI**
1. Server Manager → Add Roles and Features.
2. File and Storage Services → File and iSCSI Services.
3. Sélectionner **File Server** et **File Server Resource Manager**.
4. Installer.

**PowerShell**

```powershell
Install-WindowsFeature FS-FileServer, FS-Resource-Manager -IncludeManagementTools
Get-WindowsFeature FS-FileServer, FS-Resource-Manager
```

## 4.2 Créer un dossier partagé

Exemple : dossier `D:\Partages\Commun`.

```powershell
New-Item -Path "D:\Partages\Commun" -ItemType Directory -Force
New-SmbShare -Name "Commun" -Path "D:\Partages\Commun" `
  -ChangeAccess "ENTREPRISE\Domain Users" `
  -FullAccess "ENTREPRISE\Domain Admins"

Get-SmbShare
Get-SmbShareAccess -Name "Commun"
```

Adapter les groupes au domaine. Vérifier aussi les permissions NTFS dans l'onglet `Security`.

**Différence importante**
- Permissions de partage : appliquées lors d'un accès réseau SMB.
- Permissions NTFS : appliquées au niveau du système de fichiers, en local et via le réseau.
- Pour l'accès réseau, les droits effectifs résultent de la combinaison des permissions de partage et NTFS ; le droit le plus restrictif s'applique.

Tester depuis le client :

```text
\\SRV-FS\Commun
```

## 4.3 Configurer un quota FSRM

**GUI**
1. Ouvrir `Tools` → `File Server Resource Manager`.
2. `Quota Management` → `Quotas`.
3. `Create Quota`.
4. Choisir le chemin du dossier.
5. Choisir `Create quota on path` ou `Auto apply template` selon le scénario.
6. Choisir un modèle ou un quota personnalisé.
7. Configurer seuils et notifications selon le besoin.
8. Valider puis vérifier le quota.

**PowerShell — exemple**

```powershell
New-FsrmQuota -Path "D:\Partages\Commun" -Size 5GB
Get-FsrmQuota -Path "D:\Partages\Commun"
```

Pour une limite souple, examiner `New-FsrmQuota` et `New-FsrmAutoQuota` avec `Get-Help`; le scénario doit préciser le type attendu.

## 4.4 Créer un filtrage de fichiers

**GUI**
1. FSRM → `File Screening Management` → `File Screens`.
2. `Create File Screen`.
3. Chemin : `D:\Partages\Commun`.
4. Choisir un modèle de filtrage ou un filtrage personnalisé.
5. Vérifier les groupes de fichiers inclus.
6. Choisir le type actif/passif selon l'exercice.
7. Configurer les notifications si demandées.
8. Valider puis tester avec un fichier autorisé et un fichier bloqué.

**PowerShell — commencer par examiner les groupes existants :**

```powershell
Get-FsrmFileGroup
Get-FsrmFileScreen
```

Les noms des groupes intégrés peuvent dépendre de la langue/version. Utiliser le nom exact affiché par `Get-FsrmFileGroup`.

Exemple de création d'un groupe personnalisé puis d'un écran (adapter les extensions et le nom) :

```powershell
New-FsrmFileGroup -Name "Fichiers-Interdits-Lab" `
  -IncludePattern @("*.mp3","*.mp4","*.iso")

New-FsrmFileScreen -Path "D:\Partages\Commun" `
  -IncludeGroup "Fichiers-Interdits-Lab"
```

Tester dans le dossier surveillé. Un écran actif bloque les fichiers correspondants ; un écran passif journalise/alerte selon sa configuration.

## 4.5 Présenter la déduplication des données

Le référentiel demande de présenter cette fonctionnalité. Elle réduit l'espace utilisé en identifiant et en stockant une seule copie des blocs de données identiques.

**GUI**
1. Installer la fonctionnalité `Data Deduplication` si disponible.
2. Server Manager → File and Storage Services → Volumes.
3. Sélectionner le volume → `Configure Data Deduplication`.
4. Choisir le type d'utilisation adapté.
5. Définir l'âge minimal des fichiers et planifier l'optimisation.
6. Surveiller l'état et vérifier que le volume est éligible.

**PowerShell — exemple à faire sur un volume de lab compatible**

```powershell
Install-WindowsFeature FS-Data-Deduplication -IncludeManagementTools
Enable-DedupVolume -Volume "D:" -UsageType Default
Get-DedupVolume
```

Ne pas activer sur un volume système ou un volume non pris en charge. Vérifier la compatibilité de la version Windows Server et du type de volume.

## 4.6 Storage Spaces — créer un espace de stockage et un disque virtuel

**Le référentiel demande une configuration graphique des disques virtuels dans un espace de stockage.**

Prérequis : plusieurs disques virtuels dédiés au lab, non utilisés par le système. La création d'un pool peut effacer les données des disques sélectionnés.

**GUI**
1. Server Manager → `File and Storage Services` → `Storage Pools`.
2. `Tasks` → `New Storage Pool`.
3. Sélectionner le serveur et les disques physiques disponibles.
4. Donner un nom au pool et créer.
5. Dans le pool, créer un `Virtual Disk`.
6. Choisir le layout disponible : Simple, Mirror ou Parity selon les disques et l'objectif.
7. Choisir le provisioning (thin/thick) si demandé.
8. Créer le disque virtuel.
9. Créer un nouveau volume, choisir lettre, système de fichiers et étiquette.
10. Vérifier dans Disk Management et File and Storage Services.

**À connaître**
- Simple : performance/capacité, sans résilience.
- Mirror : copies redondantes, nécessite plusieurs disques.
- Parity : parité, compromis capacité/résilience, nécessite plusieurs disques.
- Les possibilités exactes dépendent du nombre de disques et de la version.

### Validation FSRM / fichiers

- [ ] Rôle File Server installé.
- [ ] Dossier partagé créé.
- [ ] Permissions de partage configurées.
- [ ] Permissions NTFS configurées.
- [ ] Accès testé depuis le client.
- [ ] Quota créé et vérifié.
- [ ] File Screen créé et testé.
- [ ] Déduplication présentée/configurée dans un volume de lab compatible.
- [ ] Storage Pool, Virtual Disk et volume créés graphiquement.

---

# 5. DFS — Espace de noms et réplication

## 5.1 Comprendre DFS

- **DFS Namespace (DFS-N)** : fournit un chemin logique unique vers des dossiers partagés, même si les dossiers sont hébergés sur plusieurs serveurs.
- **DFS Replication (DFS-R)** : réplique le contenu entre dossiers sur plusieurs serveurs.
- Espace de noms autonome : hébergé par un serveur.
- Espace de noms basé sur un domaine : intégré à l'espace de noms du domaine et accessible via un chemin de type `\\entreprise.local\Fichiers`.

## 5.2 Installer DFS Namespace et DFS Replication

**GUI**
1. Server Manager → Add Roles and Features.
2. File and Storage Services → File and iSCSI Services.
3. Cocher **DFS Namespaces** et **DFS Replication**.
4. Installer sur les serveurs concernés.

**PowerShell**

```powershell
Install-WindowsFeature FS-DFS-Namespace, FS-DFS-Replication -IncludeManagementTools
Get-WindowsFeature FS-DFS-Namespace, FS-DFS-Replication
```

## 5.3 Préparer les dossiers cibles

Sur SRV-FS et un second serveur (ex. SRV-FS2), créer un dossier et un partage sur chaque serveur. Exemple :

```powershell
New-Item -Path "D:\DFSData\Commun" -ItemType Directory -Force
New-SmbShare -Name "Commun" -Path "D:\DFSData\Commun" `
  -ChangeAccess "ENTREPRISE\Domain Users" `
  -FullAccess "ENTREPRISE\Domain Admins"
```

Créer également le partage correspondant sur SRV-FS2. Vérifier les accès SMB avant de créer DFS.

## 5.4 Créer un espace de noms basé sur le domaine

**GUI**
1. Ouvrir `DFS Management` (`dfsmgmt.msc`).
2. Clic droit `Namespaces` → `New Namespace`.
3. Serveur d'hébergement : `SRV-FS`.
4. Nom : `Fichiers`.
5. Choisir `Domain-based namespace`.
6. Choisir le mode approprié selon le laboratoire.
7. Terminer.

Résultat attendu :

```text
\\entreprise.local\Fichiers
```

**PowerShell — exemple**

```powershell
New-DfsnRoot -TargetPath "\\SRV-FS\Fichiers$" `
  -Path "\\entreprise.local\Fichiers" -Type DomainV2
```

Le partage cible `Fichiers$` doit exister avant la commande.

## 5.5 Ajouter un dossier DFS et ses cibles

Dans DFS Management :
1. Clic droit sur l'espace de noms → `New Folder`.
2. Nom du dossier : `Commun`.
3. Ajouter les cibles :
   - `\\SRV-FS\Commun`
   - `\\SRV-FS2\Commun`
4. Activer la référence aux cibles selon le scénario.
5. Tester le chemin DFS depuis le client.

**PowerShell — exemple**

```powershell
New-DfsnFolder -Path "\\entreprise.local\Fichiers\Commun" `
  -TargetPath "\\SRV-FS\Commun"

New-DfsnFolderTarget -Path "\\entreprise.local\Fichiers\Commun" `
  -TargetPath "\\SRV-FS2\Commun"
```

## 5.6 Configurer DFS Replication

**GUI (méthode recommandée pour l'exercice)**
1. Ouvrir DFS Management.
2. Clic droit `Replication` → `New Replication Group`.
3. Choisir `Multipurpose replication group`.
4. Nommer le groupe (ex. `RG-Commun`).
5. Ajouter SRV-FS et SRV-FS2 comme membres.
6. Choisir la topologie (Full Mesh si adaptée au nombre de serveurs).
7. Définir la planification et la bande passante.
8. Désigner le membre principal initial si l'assistant le demande.
9. Définir les chemins locaux répliqués sur chaque membre.
10. Terminer et attendre la réplication initiale.

**Tests**
1. Créer un fichier de test dans le dossier répliqué de SRV-FS.
2. Attendre la réplication.
3. Vérifier la présence du fichier sur SRV-FS2.
4. Tester l'accès via `\\entreprise.local\Fichiers\Commun`.
5. Vérifier l'état dans DFS Management.

> Éviter de modifier simultanément le même fichier sur plusieurs membres pendant le test. DFS-R n'est pas un système de verrouillage collaboratif et ne remplace pas une sauvegarde.

### Validation DFS

- [ ] DFS Namespace installé.
- [ ] DFS Replication installé.
- [ ] Espace de noms créé.
- [ ] Dossier DFS créé.
- [ ] Deux cibles ajoutées.
- [ ] Groupe de réplication configuré.
- [ ] Fichier test répliqué.
- [ ] Accès client via le chemin logique vérifié.

---

# 6. WDS — Installation, images, PXE, capture et multicast

## 6.1 Préparer WDS

Prérequis usuels :
- serveur Windows Server compatible ;
- réseau de lab isolé ;
- DHCP fonctionnel (ou configuration réseau adaptée si DHCP est ailleurs) ;
- DNS fonctionnel dans un domaine ;
- image Windows et fichiers d'installation provenant d'une source autorisée ;
- espace disque suffisant.

WDS déploie Windows via le réseau. PXE permet au client de démarrer depuis le réseau.

## 6.2 Installer le rôle WDS

**GUI**
1. Server Manager → Add Roles and Features.
2. Cocher **Windows Deployment Services**.
3. Sélectionner Deployment Server et Transport Server si l'assistant le demande.
4. Installer.
5. Ouvrir `Windows Deployment Services`.

**PowerShell**

```powershell
Install-WindowsFeature WDS -IncludeManagementTools
Get-WindowsFeature WDS
```

## 6.3 Configurer le serveur WDS

**GUI**
1. Ouvrir WDS.
2. Clic droit sur le serveur → `Configure Server`.
3. Choisir l'emplacement du dossier RemoteInstall sur un volume adapté.
4. Choisir l'intégration AD ou le mode autonome selon le lab.
5. Configurer la réponse PXE aux clients connus/inconnus conformément aux consignes de sécurité du lab.
6. Terminer puis démarrer le service.

Ne pas autoriser sans contrôle tous les clients PXE sur un réseau de production.

## 6.4 Ajouter une image de démarrage

1. WDS → `Boot Images`.
2. Clic droit → `Add Boot Image`.
3. Sélectionner `boot.wim` depuis un média Windows autorisé (souvent dans `sources`).
4. Donner un nom et une description.
5. Terminer.

## 6.5 Ajouter une image d'installation

1. WDS → `Install Images`.
2. Créer un groupe d'images (ex. `Windows-Lab`).
3. Clic droit sur le groupe → `Add Install Image`.
4. Sélectionner `install.wim` ou une image compatible.
5. Sélectionner l'édition demandée.
6. Terminer.

## 6.6 Démarrer un client par PXE

1. Vérifier que le client est connecté au bon réseau virtuel.
2. Activer le démarrage réseau/PXE dans les paramètres de la VM.
3. Démarrer le client et choisir le démarrage réseau.
4. Vérifier qu'il obtient une adresse DHCP.
5. Appuyer sur la touche demandée pour démarrer depuis le réseau.
6. Choisir l'image de démarrage WDS.
7. S'authentifier si demandé.
8. Choisir l'image d'installation et suivre l'assistant.

Si PXE échoue, vérifier le VLAN/réseau virtuel, DHCP, DNS, pare-feu, services WDS et paramètres firmware UEFI/BIOS.

## 6.7 Créer une image de capture et capturer un poste de référence

**Concepts**
- Poste de référence : machine préparée avec Windows, mises à jour et logiciels souhaités.
- Image de capture : image de démarrage spéciale qui lance l'assistant de capture.
- Image capturée : image personnalisée du poste de référence, ensuite importée dans WDS.

**Étapes GUI**
1. Dans WDS → `Boot Images`, clic droit sur l'image de démarrage de référence.
2. Choisir `Create Capture Image`.
3. Nommer l'image et enregistrer le fichier généré.
4. Ajouter cette image de capture à `Boot Images` dans WDS.
5. Préparer le poste de référence selon le scénario (généralement avec Sysprep si requis).
6. Démarrer le poste de référence par PXE.
7. Sélectionner l'image de capture.
8. Suivre l'assistant, choisir le volume Windows et le serveur/groupe d'images de destination.
9. Terminer la capture puis vérifier le fichier obtenu.
10. Ajouter l'image capturée à un groupe `Install Images`.

Respecter les consignes du formateur concernant Sysprep, les pilotes et l'édition Windows. Ne pas capturer une machine contenant des données personnelles.

## 6.8 Déploiement multicast

1. Vérifier que l'environnement réseau prend en charge le multicast.
2. Dans WDS, créer une transmission multicast.
3. Sélectionner le groupe d'images d'installation.
4. Choisir le mode de transmission demandé (automatique ou planifié, selon les options disponibles).
5. Démarrer plusieurs clients PXE et les inscrire à la transmission.
6. Surveiller la progression.

Le multicast distribue une image à plusieurs clients en limitant les transmissions redondantes. Il nécessite une infrastructure réseau adaptée ; ne pas supposer qu'il fonctionne sur tous les réseaux virtuels.

## 6.9 Fichier de réponse unattend.xml

Le fichier `unattend.xml` automatise certaines étapes de l'installation Windows (paramètres régionaux, disque, nom machine, etc.).

À savoir pour l'examen :
- il est structuré en passes de configuration ;
- les paramètres doivent correspondre à l'architecture et à la version Windows ;
- il peut contenir des informations sensibles : ne pas y stocker de mots de passe en clair ;
- utiliser Windows System Image Manager (Windows ADK) pour créer/valider un fichier de réponse adapté.

### Validation WDS

- [ ] Rôle WDS installé.
- [ ] Serveur WDS configuré et service opérationnel.
- [ ] Image de démarrage ajoutée.
- [ ] Image d'installation ajoutée.
- [ ] Client de lab démarre en PXE.
- [ ] Image de capture créée et ajoutée.
- [ ] Poste de référence capturé.
- [ ] Image capturée ajoutée comme image d'installation.
- [ ] Principe et prérequis du multicast expliqués/testés.
- [ ] Structure et rôle de `unattend.xml` expliqués.

---

# 7. Exercices pratiques type EFM

Réaliser les exercices sans regarder le corrigé, puis vérifier avec les commandes.

## Exercice 1 — DHCP (configuration complète)

**Énoncé**
Sur SRV-DC, installer DHCP et créer l'étendue `LAN-Entreprise` :
- réseau `192.168.10.0/24` ;
- plage `192.168.10.100–192.168.10.200` ;
- exclusion `192.168.10.100–192.168.10.109` ;
- routeur `192.168.10.1` ;
- DNS `192.168.10.10` ;
- domaine `entreprise.local`.
Réserver `192.168.10.50` pour une imprimante MAC `00-0C-29-8E-B5-82`. Tester depuis Client-01 et sauvegarder la base.

**Résultat attendu**
- Client-01 reçoit une IP dans la plage non exclue.
- Les options réseau sont correctes.
- La réservation apparaît dans la console DHCP.
- La sauvegarde existe dans le dossier choisi.

## Exercice 2 — DNS (zones et enregistrements)

**Énoncé**
Installer DNS. Créer la zone directe `entreprise.local`, la zone inversée du réseau `192.168.10.0/24`, puis :
- A `srv-fs` → `192.168.10.20` ;
- CNAME `fichiers` → `srv-fs.entreprise.local` ;
- PTR pour `192.168.10.20`.
Créer une zone secondaire sur DNS2 et vérifier le transfert. Tester la résolution directe et inverse.

**Résultat attendu**
```powershell
Resolve-DnsName srv-fs.entreprise.local
Resolve-DnsName fichiers.entreprise.local
Resolve-DnsName 192.168.10.20
```

## Exercice 3 — GPO

**Énoncé**
Créer l'OU `OU-Lab`, créer le GPO `GPO-LAB-Restriction`, le lier à l'OU, configurer un paramètre non destructif, appliquer un filtrage de sécurité, puis vérifier sur le client avec `gpresult`.

**Résultat attendu**
- Le GPO est lié à la bonne OU.
- Le compte/machine ciblé reçoit le paramètre.
- Un compte non ciblé ne reçoit pas le GPO.
- Le rapport HTML explique les GPO appliqués et refusés.

## Exercice 4 — PSO

**Énoncé**
Créer une PSO `PSO-Admins-Lab` avec longueur minimale 12, complexité activée, historique 10, verrouillage après 5 tentatives, puis l'appliquer à un groupe de test. Vérifier la stratégie résultante d'un utilisateur du groupe.

**Résultat attendu**
```powershell
Get-ADUserResultantPasswordPolicy -Identity "Test-GPO"
```

## Exercice 5 — FSRM

**Énoncé**
Installer File Server et FSRM. Créer `D:\Partages\Commun`, le partager, définir les permissions, créer un quota de 5 Go et un File Screen qui bloque les extensions `.mp3`, `.mp4` et `.iso`. Tester depuis un client.

**Résultat attendu**
- L'utilisateur autorisé peut accéder au partage.
- L'utilisateur non autorisé ne peut pas accéder.
- Le quota est visible dans FSRM.
- Les extensions filtrées sont bloquées si l'écran est actif.

## Exercice 6 — Storage Spaces

**Énoncé**
À l'aide de Server Manager, créer un pool avec les disques virtuels dédiés, créer un disque virtuel en miroir si le nombre de disques le permet, puis créer et formater un volume.

**Résultat attendu**
- Pool affiché dans Storage Pools.
- Disque virtuel affiché.
- Volume monté avec une lettre et un système de fichiers.
- Expliquer le choix du layout.

## Exercice 7 — DFS

**Énoncé**
Installer DFS-N et DFS-R sur SRV-FS et SRV-FS2. Créer `\\entreprise.local\Fichiers`, ajouter le dossier `Commun` avec deux cibles, configurer un groupe de réplication et tester la copie d'un fichier.

**Résultat attendu**
- Le chemin DFS est accessible depuis Client-01.
- Les deux cibles sont visibles.
- Le fichier test se réplique sur le second serveur.

## Exercice 8 — WDS

**Énoncé**
Installer et configurer WDS, ajouter une image de démarrage et une image d'installation, démarrer un client en PXE, créer une image de capture, capturer un poste de référence, puis expliquer le multicast et `unattend.xml`.

**Résultat attendu**
- Le client démarre via PXE.
- Les images sont visibles dans WDS.
- Le processus de capture est maîtrisé.
- Les conditions nécessaires au multicast sont expliquées.

---

# 8. Dépannage rapide

| Symptôme | Vérifications |
|---|---|
| Client ne reçoit pas d'IP | DHCP autorisé, étendue active, plage disponible, réseau/VLAN, service DHCP |
| DNS ne résout pas le nom | DNS client configuré, enregistrement présent, zone chargée, `nslookup`, pare-feu |
| GPO non appliqué | OU correcte, lien actif, filtrage, héritage, `gpupdate`, `gpresult` |
| Accès au partage refusé | chemin SMB, permissions de partage ET NTFS, groupe utilisateur, session |
| Quota/filtrage non effectif | FSRM installé, chemin exact, quota/file screen actif, groupe de fichiers |
| DFS inaccessible | namespace cible, DNS/AD, partages cibles, permissions |
| DFS-R ne réplique pas | service DFSR, membres, dossier répliqué, connectivité, état de réplication |
| PXE ne démarre pas | DHCP, réseau virtuel/VLAN, UEFI/BIOS, WDS, boot image, pare-feu |

Commandes de diagnostic utiles :

```powershell
ipconfig /all
Test-NetConnection SRV-DC -Port 53
Test-NetConnection SRV-DC -Port 67
Resolve-DnsName entreprise.local
Get-Service DHCPServer,DNS,DFSR -ErrorAction SilentlyContinue
```

`Test-NetConnection` vérifie un port TCP ; DHCP utilise aussi UDP, donc ce test seul ne prouve pas le bon fonctionnement DHCP.

---

# 9. QCM de contrôle rapide

**1. Quel est le rôle principal de DHCP ?**
- A. Résoudre les noms
- B. Attribuer automatiquement les paramètres IP
- C. Répliquer les fichiers
- D. Déployer les images Windows

**2. Quel enregistrement DNS associe un nom à une adresse IPv4 ?**
- A. PTR
- B. CNAME
- C. A
- D. MX

**3. Quel ordre représente l'application standard des GPO ?**
- A. LSDOU
- B. DOLSU
- C. OU-LSD
- D. SLUOD

**4. Quel outil sert à gérer les GPO ?**
- A. DNS Manager
- B. GPMC
- C. DHCP Manager
- D. WDS Console

**5. FSRM permet notamment de :**
- A. Créer des comptes AD
- B. Gérer quotas et filtrage de fichiers
- C. Attribuer les adresses IP
- D. Résoudre les noms DNS

**6. DFS Namespace fournit :**
- A. Un chemin logique vers des partages
- B. Une adresse IP automatique
- C. Un serveur DHCP secondaire
- D. Une image de démarrage PXE

**7. DFS-R sert à :**
- A. Répliquer des dossiers
- B. Créer des GPO
- C. Configurer les zones DNS
- D. Sauvegarder DHCP

**8. PXE est utilisé pour :**
- A. Démarrer un ordinateur depuis le réseau
- B. Créer un quota
- C. Modifier un GPO
- D. Créer une zone inversée

**9. Une réservation DHCP associe généralement :**
- A. Un nom DNS à un CNAME
- B. Un identifiant client à une adresse IP
- C. Une OU à un GPO
- D. Un dossier à un quota

**10. Une zone DNS secondaire est généralement :**
- A. Une copie en lecture seule obtenue par transfert
- B. Une zone qui ne contient que des PTR
- C. Une zone DHCP
- D. Un type de GPO

**Corrigé :** 1-B, 2-C, 3-A, 4-B, 5-B, 6-A, 7-A, 8-A, 9-B, 10-A.

---

# 10. Checklist finale EFM

- [ ] Je peux installer les rôles DHCP, DNS, File Server, FSRM, DFS et WDS.
- [ ] Je peux créer une étendue DHCP, ses options et une réservation.
- [ ] Je peux sauvegarder/restaurer DHCP et expliquer la haute disponibilité.
- [ ] Je peux créer des zones DNS directes, inversées, principales et secondaires.
- [ ] Je peux créer A, AAAA, PTR et CNAME.
- [ ] Je peux tester le DNS, les redirecteurs, le nettoyage et expliquer une politique DNS.
- [ ] Je peux créer, lier, modifier et supprimer un lien GPO.
- [ ] Je comprends LSDOU, filtrage de sécurité, WMI, Block Inheritance et Enforced.
- [ ] Je peux créer une PSO et vérifier la stratégie résultante.
- [ ] Je peux créer un partage et configurer les droits SMB/NTFS.
- [ ] Je peux créer un quota et un File Screen FSRM.
- [ ] Je peux expliquer la déduplication et créer un Storage Space via GUI.
- [ ] Je peux créer un namespace DFS, ajouter des cibles et tester DFS-R.
- [ ] Je peux configurer WDS, ajouter les images et démarrer un client PXE.
- [ ] Je comprends la capture, le multicast et `unattend.xml`.
- [ ] Je sais vérifier le résultat et diagnostiquer les erreurs de base.

---

## Référence au référentiel OFPPT

Ce guide suit les objectifs pratiques indiqués dans le PDF source :
- **D (p. 8–9)** : DHCP, étendue, options, réservation, sauvegarde/restauration, PowerShell.
- **E (p. 9–11)** : DNS, zones directes/inversées/principales/secondaires, A/AAAA/PTR/CNAME, scavenging, politique DNS, PowerShell.
- **F (p. 11–12)** : GPO, liaison, suppression de liaison, filtres, héritage, Central Store, PSO, PowerShell.
- **G (p. 12–13)** : espace de stockage, partage, quotas, filtrage, déduplication.
- **H (p. 13–14)** : DFS Namespace, configuration et scénarios de réplication.
- **I (p. 14)** : WDS, images de démarrage/installation, capture, poste de référence, multicast et `unattend.xml`.

**Conseil de révision :** exécuter chaque TP au moins une fois avec l'interface graphique, puis refaire les opérations pour lesquelles le référentiel demande PowerShell. Garder des captures d'écran des résultats et noter les erreurs rencontrées.
