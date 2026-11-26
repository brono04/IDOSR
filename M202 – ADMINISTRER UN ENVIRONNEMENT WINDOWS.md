### 🧠 Réponses courtes à mémoriser

#### 🖥️ Types de Windows Server
1. **Standard** → pour les besoins classiques 🏢
2. **Datacenter** → pour les environnements virtualisés et datacenters 🏭
3. **Essentials** → pour les petites entreprises 👥

#### ⚙️ Types d’installation

- **Server Core** → installation minimale
- **Desktop Experience** → installation complète avec interface graphique

1. **DC** → Serveur qui gère le domaine et authentifie les utilisateurs.
    
2. **Group** → Ensemble d’utilisateurs ou d’ordinateurs pour gérer les permissions.
    
3. **Global Catalog** → Catalogue des objets de la forêt AD.
    
4. **DNS** → Résout les noms et permet de trouver les DC.
    
5. **Authorization** → Détermine le doit de utilisateur.
    
6. **Server Manager** → Outil pour gérer les serveurs Windows.
    
7. **Gestion centralisée** → Installer AD DS → créer le domaine → promouvoir en DC.
    
8. **Structure AD** → Domaine → OU → Users/Computers.
# Gestion du Cycle de Vie d'Active Directory via PowerShell
## 🛠️ Étapes de Déploiement

**Directory Services Restore Mode (DSRM)**:un mode de démarrage spécial  de  Windows Server, utilisé pour réparer ou restaurer la base de données Active Directory (AD).
- **Installer AD DS** copie simplement les **fichiers logiciels** sur le serveur. Rien ne change pour l'instant.
- **Promouvoir le serveur** le transforme en **Contrôleur de Domaine**. C'est là qu'il crée la base de données et configure le DNS.
Rétrogradation : Supprime  le contrôleur de domaine de l'AD et du DNS.

| Caractéristique | Groupe de travail (Workgroup)             | Domaine (Domain)                          |
| --------------- | ----------------------------------------- | ----------------------------------------- |
| Gestion         | Indépendante (chaque PC se gère lui-même) | Centralisée (gérée par un serveur unique) |
| Connexion       | Un compte sur chaque PC                   | Un seul compte pour tous les PC           |
| Idéal pour      | Maison et petites entreprises (< 20 PC)   | Entreprises et grands réseaux             |
| Sécurité        | Faible (à configurer sur chaque PC)       | Forte (contrôlée par le serveur)          |
# Active Directory – Notes PowerShell

> **DSRM** (Directory Services Restore Mode) → mode spécial pour réparer/restaurer AD.

## 1. Installation

**Installer AD DS** → installe les fichiers et outils AD.

```powershell
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
```

**Promouvoir le serveur** → devient contrôleur de domaine + crée la forêt/DNS.

```powershell
Install-ADDSForest -DomainName "formation.local"
```

**Vérifier**

```powershell
Get-ADDomain
Get-ADForest
$env:USERDNSDOMAIN
```

## 🛑 Suppression

**Rétrograder** → retire le contrôleur de domaine d'AD/DNS.

```powershell
Uninstall-ADDSDomainController
```

**Désinstaller AD DS**

```powershell
Uninstall-WindowsFeature AD-Domain-Services
```

## 2. 📂 OU (Organizational Unit)

OU = conteneur pour organiser Users/Computers.

```powershell
# Créer
New-ADOrganizationalUnit -Name "DRRabat" -Path "DC=formation,DC=local"

# Chercher (DR* = commence par DR)
Get-ADOrganizationalUnit -Filter 'Name -like "DR*"'

# Modifier
Set-ADOrganizationalUnit -Identity "OU=DRCasa,DC=formation,DC=local" -Description "..."

# Déplacer (Rabat → Casa)
Move-ADObject -Identity "OU=RH,OU=DRRabat,DC=formation,DC=local" -TargetPath "OU=DRCasa,DC=formation,DC=local"

# Supprimer (désactiver d'abord la protection)
Set-ADOrganizationalUnit -Identity "OU=RH,OU=DRCasa,DC=formation,DC=local" -ProtectedFromAccidentalDeletion $false
Remove-ADOrganizationalUnit -Identity "OU=RH,OU=DRCasa,DC=formation,DC=local"
```
## 3. 👤 Users

```powershell
# Créer
New-ADUser -Name "John Robinson" -SamAccountName "John.Robinson"

# Chercher
Get-ADUser -Filter *

# Modifier
Set-ADUser John.Robinson -EmailAddress "john@formation.local"

# Supprimer
Remove-ADUser John.Robinson

# Déplacer
Move-ADObject -Identity "CN=John Robinson,OU=IDOSR,DC=formation,DC=local" -TargetPath "OU=IDOSR202,DC=formation,DC=local"

# Mot de passe
Set-ADAccountPassword John.Robinson -Reset -NewPassword (Read-Host -AsSecureString "Enter new password")

# Activer / Désactiver
Enable-ADAccount John.Robinson
Disable-ADAccount John.Robinson

# Débloquer
Unlock-ADAccount John.Robinson

```

---
## 4. 👥 Groups

Groupe = ensemble d'utilisateurs/ordinateurs pour gérer les droits.

```powershell
# Créer
New-ADGroup -Name "GRH" -GroupScope Global -GroupCategory Security -Path "OU=RH,OU=DRCasa,DC=formation,DC=local"

# Chercher
Get-ADGroup -Filter 'Name -like "G*"'

# Modifier
Set-ADGroup -Identity "GRH" -Description "Groupe RH"

# Supprimer
Remove-ADGroup -Identity "GRH"

# Ajouter un membre
Add-ADGroupMember -Identity "GRH" -Members John.Robinson

# Lister les membres
Get-ADGroupMember -Identity "GRH"

# Retirer un membre
Remove-ADGroupMember -Identity "GRH" -Members John.Robinson

# Déplacer
Move-ADObject -Identity "CN=GRH,OU=RH,OU=DRCasa,DC=formation,DC=local" -TargetPath "OU=DRRabat,DC=formation,DC=local"
```

---
# 6. 💻 Computers

### Créer

```
New-ADComputer -Name "PC-RH-01" -Path "OU=DRRabat,DC=formation,DC=local"
```

### Chercher

```
Get-ADComputer -Filter *
```

### Supprimer

```
Remove-ADComputer -Identity "PC-RH-01"
```

---
# 7. 🌐 Joindre un PC au domaine

**DNS client → IP du DC**

Puis :

```
Add-Computer -DomainName "formation.local" -Restart
```

---
##  🔎 Autres méthodes de filtrage PowerShell (Active Directory)

```powershell
# 1. Commence par DR
Get-ADOrganizationalUnit -Filter 'Name -like "DR*"'

# 2. Se termine par DR
Get-ADOrganizationalUnit -Filter 'Name -like "*DR"'

# 3. Contient DR
Get-ADOrganizationalUnit -Filter 'Name -like "*DR*"'

# 4. Nom exact
Get-ADOrganizationalUnit -Filter 'Name -eq "DR"'

# 5. Nom différent de DR
Get-ADOrganizationalUnit -Filter 'Name -ne "DR"'

# 6. Nom commençant par DR ou IT
Get-ADOrganizationalUnit -Filter 'Name -like "DR*" -or Name -like "IT*"'

# 7. Nom commençant par DR et activé ? 

Get-ADUser -Filter * -Properties EmailAddress | Select-Object Name, SamAccountName, EmailAddress

# (Une OU n'a pas de propriété Enabled standard)
```

# 🌐 DHCP PowerShell — Cheat Sheet

---
## 🔧 Installation

```powershell
# 1. Installer le rôle
Install-WindowsFeature -Name DHCP -IncludeManagementTools

# 2. Autoriser dans Active Directory (si domaine AD)
Add-DhcpServerInDc -DnsName AD1.formation.local -IPAddress 192.168.1.90
```

---
## 📦 Étendue (Scope)

```powershell
# Créer
Add-DhcpServerv4Scope -Name "IT-LAN" -StartRange 192.168.2.1 -EndRange 192.168.2.254 -SubnetMask 255.255.255.0

# Durée de bail (4 jours)
Set-DhcpServerv4Scope -ScopeId 192.168.2.0 -LeaseDuration 4.00:00:00

# Exclure des adresses
Add-DhcpServerv4ExclusionRange -ScopeId 192.168.2.0 -StartRange 192.168.2.10 -EndRange 192.168.2.20

# Options (passerelle, DNS, WINS, domaine)
Set-DhcpServerv4OptionValue -ScopeId 192.168.2.0 -Router 192.168.2.254 -DnsServer 192.168.1.90 -WinsServer 192.168.1.2 -DnsDomain formation.local

# Activer ✅
Set-DhcpServerv4Scope -ScopeId 192.168.2.0 -Name "IT-LAN" -State Active
```

> ⚠️ Sans `-ScopeId`, les options s'appliquent à **tout le serveur**.

---

## 📋 Afficher / Supprimer

```powershell
Get-DhcpServerv4Scope                          # Voir les étendues
Get-DhcpServerv4OptionValue -ScopeId 192.168.2.0  # Voir les options
Remove-DhcpServerv4Scope -ScopeId 192.168.2.0 -Force  # Supprimer
```

---
## 📌 Réservation (IP fixe par MAC)

```powershell
Add-DhcpServerv4Reservation `
  -ScopeId 192.168.2.0 `
  -Name "CLIENT" `
  -IPAddress 192.168.2.101 `
  -ClientId "00-0c-29-8e-b5-82" `
  -Description "Réservation pour la machine CLIENT"
```

---
## ⚡ Ordre de configuration

```
1. Install-WindowsFeature
2. Add-DhcpServerInDc         (AD uniquement)
3. Add-DhcpServerv4Scope      → créer
4. Set-DhcpServerv4Scope      → durée de bail
5. Add-DhcpServerv4ExclusionRange → exclure
6. Set-DhcpServerv4OptionValue    → options réseau
7. Set-DhcpServerv4Scope -State Active  ✅
8. Add-DhcpServerv4Reservation    → réservations MAC
```
# 🌐 DNS via PowerShell — Fiche de révision

## 1. Installer DNS

```powershell
Install-WindowsFeature -Name DNS -IncludeManagementTools
```

## 2. Créer une zone directe

- **Intégrée à AD :**

```powershell
Add-DnsServerPrimaryZone -Name "idosr.local" -ReplicationScope Domain -DynamicUpdate Secure
```

- **Non intégrée à AD :**

```powershell
Add-DnsServerPrimaryZone -Name "idosr.local" -ZoneFile "idosr.local.dns"
```

## 3. Créer une zone inversée

- **Intégrée à AD :**

```powershell
Add-DnsServerPrimaryZone -NetworkID "192.168.2.0/24" -ReplicationScope Domain -DynamicUpdate Secure
```

- **Non intégrée à AD :**

```powershell
Add-DnsServerPrimaryZone -NetworkID "192.168.2.0/24" -ZoneFile "2.168.192.in-addr.arpa.dns"
```

## 4. Gérer les zones

```powershell
Get-DnsServerZone
Remove-DnsServerZone -ZoneName "idosr.local"
```

## 5. Enregistrements DNS

- **A** : nom → adresse IPv4
- **CNAME** : alias → nom d’hôte
- **PTR** : adresse IP → nom d’hôte
- **MX** : serveur de messagerie

```powershell
# A
Add-DnsServerResourceRecordA -Name "pc1" -ZoneName "idosr.local" -IPv4Address "192.168.2.200" -CreatePtr

# CNAME
Add-DnsServerResourceRecordCName -Name "www" -HostNameAlias "ServeurWeb.idosr.local" -ZoneName "idosr.local"

# PTR
Add-DnsServerResourceRecordPTR -Name "101" -PtrDomainName "pc2.idosr.local" -ZoneName "2.168.192.in-addr.arpa"

# MX
Add-DnsServerResourceRecordMX -Name "mail" -MailExchange "ServeurMail.idosr.local" -Preference 10 -ZoneName "idosr.local"

# Supprimer un enregistrement A
Remove-DnsServerResourceRecord -Name "pc1" -RRType "A" -ZoneName "idosr.local"
```

## 6. Afficher les enregistrements

```powershell
Get-DnsServerResourceRecord -ZoneName "idosr.local"
Get-DnsServerResourceRecord -ZoneName "idosr.local" -Name "www"
```
# **GPO**

```powershell
# Lister les GPO du domaine
Get-GPO -All

# Afficher les informations d’une GPO
Get-GPO -Name "Nom de la GPO"

# Créer une GPO
New-GPO -Name "GPO-Test"

# Lier une GPO à une OU (adapter le DN)
New-GPLink -Name "GPO-Test" -Target "OU=Utilisateurs,DC=formation,DC=local"

# Actualiser les stratégies sur l’ordinateur local
gpupdate /force
gpupdate /target:user or computer

# Afficher les stratégies appliquées à l’utilisateur et à l’ordinateur
gpresult /r

# Générer un rapport HTML des stratégies résultantes
gpresult /h "$env:USERPROFILE\Desktop\rapport-gpo.html"

# Lister les PSO
Get-ADFineGrainedPasswordPolicy -Filter *

# Afficher un PSO
Get-ADFineGrainedPasswordPolicy -Identity "NomDuPSO"

# Lister les PSO résultants d’un utilisateur
Get-ADUserResultantPasswordPolicy -Identity "utilisateur"
```

