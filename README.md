# ☁️ Déploiement d'un Cloud Privé IaaS / PaaS / SaaS avec Proxmox VE

> Travail Pratique — Cloud Computing | ING2 · IAI Gabon · 2025–2026

---

## 📋 Présentation

Ce projet documente la conception, le déploiement et la mise en service d'un **cloud privé complet** offrant les trois niveaux de services cloud :

| Couche | Service | Technologie | VM |
|--------|---------|-------------|-----|
| **IaaS** | Infrastructure as a Service | Ubuntu Server 26.04 LTS | `backupserver` (VM 100) |
| **PaaS** | Platform as a Service | Docker, MySQL, Python3, Git | `devserver` (VM 101) |
| **SaaS** | Software as a Service | Moodle 4.5.11+ | `appserver` (VM 102) |

L'ensemble repose sur **Proxmox VE**, hyperviseur open source de type 1 basé sur Debian Linux.

---

## 🛠️ Stack technique

- **Hyperviseur :** Proxmox VE 9.1.1 (KVM + LXC)
- **OS des VMs :** Ubuntu Server 26.04 LTS
- **Réseau :** Bridge `vmbr0`, IP statique `192.168.1.158`
- **Serveur physique :** Intel Core i7, 16 Go RAM, SSD externe 1 To

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────┐
│              Proxmox VE — nœud "tpcloud"            │
│                   192.168.1.158:8006                │
│                                                     │
│  ┌──────────────┐  ┌──────────────┐  ┌───────────┐ │
│  │  VM 100      │  │  VM 101      │  │  VM 102   │ │
│  │ backupserver │  │  devserver   │  │ appserver │ │
│  │ .198 — IaaS  │  │ .135 — PaaS  │  │.102 — SaaS│ │
│  │ 2 vCPU/2 Go  │  │ 4 vCPU/4 Go  │  │           │ │
│  │ 200 GiB      │  │  50 GiB      │  │           │ │
│  └──────────────┘  └──────────────┘  └───────────┘ │
│                                                     │
│                   Bridge vmbr0                      │
└─────────────────────────────────────────────────────┘
```

---

## 🚀 Déploiement

### Prérequis

- Matériel compatible VT-x / AMD-V
- Clé USB bootable Proxmox VE
- Réseau local avec adresse IP statique disponible
- Poste client Windows avec profil réseau en mode **Privé**

### 1. Installation de Proxmox VE

```bash
# Après démarrage sur la clé USB, configurer :
# - Disque cible : /dev/sda (ext4)
# - IP : 192.168.1.158/24
# - Passerelle : 192.168.1.1
# - Hostname : tpcloud.local

# Interface web accessible à :
https://192.168.1.158:8006
```

### 2. Couche IaaS — VM `backupserver` (ID 100)

```
CPU    : 2 vCPU (x86-64-v2-AES)
RAM    : 2 048 MiB
Disque : 200 GiB SCSI (VirtIO, local-lvm)
Réseau : VirtIO sur vmbr0
IP     : 192.168.1.198
```

```bash
# Connexion SSH depuis le poste client
ssh user01@192.168.1.198
```

### 3. Couche PaaS — VM `devserver` (ID 101)

```
CPU    : 4 vCPU (x86-64-v2-AES)
RAM    : 4 096 MiB
Disque : 50 GiB SCSI (VirtIO, local-lvm)
IP     : 192.168.1.135
```

Outils installés sur la VM :

```bash
sudo apt install mysql-server -y
sudo apt install docker.io -y
sudo apt install python3 python3-pip -y
sudo apt install git -y
```

### 4. Couche SaaS — Moodle sur `appserver` (VM 102)

Stack : **Apache2 + PHP + MariaDB/MySQL**

```
URL d'accès : http://192.168.1.102/moodle
Nom du site : e-iai
Version     : Moodle 4.5.11+
Base de données : appserver (utilisateur : appuser, port 3306)
```

---

## 📦 Services déployés

### IaaS
- Machines virtuelles Ubuntu Server provisionnées à la demande via l'interface Proxmox
- Accès SSH sécurisé
- Snapshots de sauvegarde (ex. `avant_samba`)

### PaaS
- Environnement de développement standardisé (Docker, MySQL, Python3, Git)
- Accessible aux développeurs/étudiants sans gestion de l'infrastructure sous-jacente
- Reverse proxy Nginx pour exposer les applications conteneurisées

### SaaS
- Plateforme e-learning **Moodle** opérationnelle
- Gestion des cours, étudiants, évaluations et ressources pédagogiques
- Accessible depuis n'importe quel navigateur du réseau local IAI

---

## 🔧 Configuration SSH (PaaS)

```bash
# Modifier /etc/ssh/sshd_config
sudo nano /etc/ssh/sshd_config
# PermitRootLogin no

# Redémarrer le service
sudo systemctl restart ssh
```

---

## 📈 Améliorations futures

- [ ] Cluster Proxmox multi-nœuds (haute disponibilité)
- [ ] Provisionnement automatisé via **Terraform** ou **Ansible**
- [ ] Authentification centralisée **LDAP**
- [ ] Monitoring (Grafana + Prometheus)
- [ ] Accès HTTPS avec certificats TLS (Let's Encrypt)

---

## 📚 Références

- [Proxmox VE Documentation](https://pve.proxmox.com/pve-docs/)
- [Docker Documentation](https://docs.docker.com/)
- [Moodle Documentation](https://docs.moodle.org/)
- [Ubuntu Server Guide](https://ubuntu.com/server/docs)
- [NIST Definition of Cloud Computing — SP 800-145](https://csrc.nist.gov/publications/detail/sp/800-145/final)
- [KVM Hypervisor](https://www.linux-kvm.org/)

---

## 📄 Licence

Projet académique — IAI Gabon · ING2 · 2025–2026  
Réalisé dans le cadre du cours de Cloud Computing.
