# Studi — TP final TSSR (Technicien Supérieur Systèmes et Réseaux, RNCP niveau 5)

> **TP final d'évaluation passée en cours de formation** — Titre Professionnel `TP-01351` (Arrêté du 26/04/2023, JO du 13/05/2023).
> Évalué le 23/12/2025 par **Julien DUBOIS** (Studi). Représentant de l'organisme : Pierre CHARVET.

## 🎯 Résultat

| | |
|---|---|
| **Note finale** | **82/100** soit **16,4/20** *(mention Bien)* |
| **Statut** | ✅ **Tous les blocs validés** |
| **Activité 1 maîtrisée** | ✅ Oui |
| **Activité 2 maîtrisée** | ✅ Oui |

> *« Bien. Tous les blocs sont validés. Attention à ne pas oublier de montrer des étapes d'installations et/ou de configurations. »* — Julien DUBOIS, évaluateur

*Livret d'évaluations officiel disponible sur demande (document République Française / Ministère du Travail).*

## Activités-types validées

### 🔧 Activité 1 — Exploiter les éléments de l'infrastructure et assurer le support aux utilisateurs

Compétences évaluées et validées :

1. ✅ Assurer le support utilisateur en centre de services
2. ✅ Exploiter des serveurs Windows et un domaine Active Directory
3. ✅ Exploiter des serveurs Linux

**Objectifs réalisés** :
- Mettre en place un système pour les rôles **AD / DNS / DHCP** (Windows Server 2022)
- Mettre en place un système de **virtualisation** dans l'entreprise (VirtualBox)
- Mettre en place une **solution de documentation** (GLPI)
- **Unification de l'authentification** sur la solution WIKIjs (via LDAP/AD)

### 🛡️ Activité 2 — Maintenir l'infrastructure et contribuer à son évolution et à sa sécurisation

Compétences évaluées et validées :

7. ✅ Maintenir et sécuriser les accès Internet et les interconnexions des réseaux
8. ✅ Mettre en place, assurer et tester les sauvegardes et les restaurations
9. ✅ Exploiter et maintenir les services de déploiement des postes de travail

**Objectifs réalisés** :
- **Déployer des stratégies de sécurité par GPO**
- **Planifier un export de base de données via crontab**
- **Automatiser la création des utilisateurs via script**

## 📋 Le scénario du TP — CreativeFusion-Studios.eu

Centralisation complète des accès, services et sauvegardes pour une PME du secteur du design/animation.

### Stack technique mise en œuvre

| Couche | Outil / Version |
|---|---|
| **Hyperviseur** | Oracle VirtualBox |
| **Contrôleur de domaine** | Windows Server 2022 (AD DS, DNS, DHCP) |
| **Serveur applicatif** | Debian 12 (sans GUI) |
| **Stack web** | Apache2 + MariaDB + PHP |
| **ITSM/Inventaire** | **GLPI 10.0.10** |
| **Authentification fédérée** | LDAP/AD |
| **Simulation réseau** | Cisco Packet Tracer |
| **Automatisation** | GPO + batch (`netsh`) |

### Plan d'adressage

| Service | Réseau | Masque |
|---|---|---|
| Ressources Humaines | 10.0.10.0 | /24 |
| Comptabilité | 10.0.20.0 | /24 |
| Informatique | 10.0.30.0 | /24 |
| Direction Générale | 10.0.40.0 | /24 |
| Design-Animation | 10.0.50.0 | /24 |
| **Serveurs** | **10.0.60.0** | /24 |

→ Forêt AD : **`CreativeFusion-Studios.eu`** — DC sur 10.0.60.10

## 🛠️ Les 7 réalisations techniques documentées

### 1. Création du domaine Active Directory (Windows Server 2022)

- Installation Windows Server 2022
- Configuration IPv4 statique sur réseau Serveurs (10.0.60.10/24)
- Promotion en contrôleur de domaine via Server Manager
- Création de la forêt **CreativeFusion-Studios.eu**
- Vérification ADUC + console DNS (zone directe avec SRV records)

### 2. Serveur DHCP (Windows Server 2022)

- Rôle DHCP Server installé + autorisé dans l'AD
- **6 étendues** créées (une par service) :
  - VLAN 10 (RH) / VLAN 20 (Comptabilité) / VLAN 30 (Informatique)
  - VLAN 40 (Direction Générale) / VLAN 50 (Design-Animation) / VLAN 60 (Serveurs)
- Options communes : **Router .254**, **DNS 10.0.60.10**, **Nom de domaine** `CreativeFusion-Studios.eu`
- Test : poste client obtient IP, gateway, DNS via DHCP

### 3. Installation GLPI 10.0.10 sur Debian 12 (sans GUI)

- Debian 12 minimal installé en VM
- IP statique sur 10.0.60.0/24
- Installation de la stack web : **Apache2 + MariaDB + PHP** + extensions GLPI
- Sécurisation MariaDB (`mysql_secure_installation`)
- Activation des services au démarrage (`systemctl enable --now`)
- Téléchargement et déploiement de **GLPI 10.0.10**
- Création BDD + utilisateur dédié + permissions
- Premier accès via l'assistant web → tableau de bord GLPI accessible

### 4. Authentification GLPI via Active Directory (LDAP)

Configuration de l'annuaire LDAP dans GLPI :

| Paramètre | Valeur |
|---|---|
| Serveur | `10.0.60.10:389` |
| Filtre de connexion | `(&(objectClass=user)(!(objectClass=Computer)))` |
| Base DN | `OU=Utilisateurs,DC=creativefusion-studios,DC=eu` |
| DN de bind | `CN=Administrateur,CN=Users,DC=creativefusion-studios,DC=eu` |
| Active | ✅ |

→ Import et synchronisation des utilisateurs depuis l'AD
→ Test de connexion GLPI réussi avec un compte de domaine (`jean.dupont`)

### 5. GPO — Script ForceDHCP au démarrage

Mission : corriger en masse des postes en IP statique → forcer IP + DNS en DHCP au démarrage.

**Script `ForceDHCP.cmd`** :

```bat
@echo off
REM Forcer la carte "Ethernet" en DHCP (adresse IP + DNS)
netsh interface ip set address name="Ethernet" source=dhcp
netsh interface ip set dns name="Ethernet" source=dhcp
```

**Déploiement** :
- Copie du script dans `SYSVOL\CreativeFusion-Studios.eu\scripts\`
- Création GPO `Force_DHCP` → Configuration ordinateur > Stratégies > Paramètres Windows > Scripts > Démarrage
- Lien sur l'OU **`Ordinateurs > Poste_clients`**

**Validation avant/après** :
- Avant `gpupdate /force` + redémarrage : `DHCP activé : Non` (IP statique 10.0.60.5)
- Après : `DHCP activé : Oui`, IP obtenue 10.0.60.6, bail expirant le 12/12/2025

### 6. Packet Tracer — VLAN + routage inter-VLAN + DHCP relay + sauvegarde FTP

**Topologie** : 1 routeur + 1 switch + plusieurs postes + serveur (DHCP/DNS/FTP)

#### Segmentation VLAN sur le switch

```cisco
vlan 10 → name RH
vlan 20 → name compta
vlan 30 → name informatique
vlan 40 → name direction
vlan 50 → name design
vlan 60 → name serveurs
```

#### Affectation des ports et trunk

```cisco
interface fa0/3-7 → switchport mode access + switchport access vlan {10|20|30|40|50}
interface fa0/2  → switchport access vlan 60 (serveur)
interface fa0/1  → switchport mode trunk (vers routeur)
```

#### Router-on-a-stick (sous-interfaces 802.1Q)

```cisco
interface g0/0.10 → encapsulation dot1Q 10 → ip address 10.0.10.254 255.255.255.0
interface g0/0.20 → encapsulation dot1Q 20 → ip address 10.0.20.254 255.255.255.0
interface g0/0.30 → encapsulation dot1Q 30 → ip address 10.0.30.254 255.255.255.0
interface g0/0.40 → encapsulation dot1Q 40 → ip address 10.0.40.254 255.255.255.0
interface g0/0.50 → encapsulation dot1Q 50 → ip address 10.0.50.254 255.255.255.0
interface g0/0.60 → encapsulation dot1Q 60 → ip address 10.0.60.254 255.255.255.0
```

#### DHCP centralisé via relais

```cisco
interface g0/0.10 → ip helper-address 10.0.60.10
interface g0/0.20 → ip helper-address 10.0.60.10
... (idem pour 30, 40, 50)
```

→ 6 pools DHCP sur le serveur, un par VLAN, avec passerelle `.254` et DNS `10.0.60.10`.

#### Tests de connectivité

- ✅ Un poste VLAN 10 obtient IP + passerelle + DNS via DHCP
- ✅ Ping inter-VLAN (10.0.20.1 depuis un PC du VLAN 10) → réponses reçues après résolution ARP

#### Sauvegarde de la configuration vers FTP

```cisco
Router# copy running-config ftp:
Address or name of remote host []? 10.0.60.10
Destination filename [Router-confg]?
Writing running-config...
[OK - 1449 bytes]
1449 bytes copied in 0.081 secs (17000 bytes/sec)
```

→ Sauvegarde de la conf du routeur vers le serveur FTP du VLAN Serveurs.

## 📂 Livrables

- 📄 [**Documentation technique complète du TP (20 pages)**](docs/02-documentation-technique-tp.pdf) — version PDF propre avec toutes les captures (AD, DHCP, GLPI, LDAP, GPO, Packet Tracer)
- 📋 *Livret d'évaluations officiel (Ministère du Travail) disponible sur demande*

## 🎓 Position de ce diplôme dans mon parcours

Ce titre **TSSR (RNCP niveau 5)** complète mon **Technicien Informatique OpenClassrooms** (RNCP niveau 5 également, code RNCP40357).

- Les deux titres sont au **niveau bac+2**
- Ils sont **complémentaires** : Studi met l'accent sur l'**infrastructure & l'admin**, OpenClassrooms sur le **support et la production**
- Combinés, ils couvrent **les 4 blocs de compétences** d'un profil ASR&C junior

🔗 **Voir le portfolio complet (13 projets OpenClassrooms)** : [github.com/PortetWilfried](https://github.com/PortetWilfried)

## Liens

- [← Retour au profil GitHub](https://github.com/PortetWilfried)
- [Portfolio Technicien Informatique OpenClassrooms (13 projets)](https://github.com/PortetWilfried)
- Fiche RNCP du titre TSSR : [France Compétences — TP-01351](https://www.francecompetences.fr/recherche/rncp/37681/)
