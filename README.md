# WSL Setup Guide

Ce guide vous permettra d'installer et configurer votre environnement WSL avec Ansible.

## Prérequis

- Windows 10/11 avec WSL2 activé
- Ubuntu installé dans WSL
- Connexion internet
- (Optionnel) VPN d'entreprise nécessitant wsl-vpnkit

## Installation de WSL

### 1. Activer WSL (PowerShell en tant qu'administrateur)

```powershell
wsl --install
```

Si WSL est déjà installé, installez Ubuntu :

```powershell
wsl --install -d Ubuntu
```

### 2. Mettre à jour WSL2

```powershell
wsl --update
wsl --set-default-version 2
```

### 3. Vérifier l'installation

```powershell
wsl --list --verbose
```

## Configuration initiale dans WSL

### 1. Mise à jour du système

```bash
sudo apt update && sudo apt upgrade -y
```

### 2. Installation d'Ansible

```bash
sudo apt install -y ansible git
```

### 3. (Optionnel) Installation de wsl-vpnkit pour VPN

Si vous utilisez un VPN d'entreprise (Cisco AnyConnect, Pulse Secure, etc.) qui bloque la connectivité réseau de WSL, installez wsl-vpnkit.

#### Sur Windows (PowerShell en tant qu'administrateur)

```powershell
# Télécharger wsl-vpnkit
$VERSION = "v0.4.1"
Invoke-WebRequest -Uri "https://github.com/sakai135/wsl-vpnkit/releases/download/$VERSION/wsl-vpnkit.tar.gz" -OutFile "$env:USERPROFILE\Downloads\wsl-vpnkit.tar.gz"

# Importer la distro wsl-vpnkit
wsl --import wsl-vpnkit --version 2 $env:USERPROFILE\wsl-vpnkit $env:USERPROFILE\Downloads\wsl-vpnkit.tar.gz

# Vérifier l'installation
wsl -d wsl-vpnkit
```

#### Dans votre distro Ubuntu WSL

Ajoutez cette ligne à votre `~/.bashrc` ou `~/.zshrc` pour démarrer automatiquement wsl-vpnkit :

```bash
# Démarrer wsl-vpnkit automatiquement
wsl.exe -d wsl-vpnkit --cd /app service wsl-vpnkit start 2>/dev/null
```

Rechargez votre configuration :

```bash
source ~/.bashrc  # ou source ~/.zshrc si vous utilisez zsh
```

**Vérifier que wsl-vpnkit fonctionne :**

```bash
# Depuis PowerShell
wsl.exe -d wsl-vpnkit --cd /app service wsl-vpnkit status

# Depuis Ubuntu WSL
ping google.com
nslookup google.com
```

### 4. Cloner votre repository

```bash
git clone https://github.com/yohandion06/setup_wsl
cd setup_wsl/ansible
```

## Nettoyage des anciennes configurations (si nécessaire)

Si vous avez déjà des configurations Docker ou Terraform existantes, nettoyez-les avant de lancer le playbook :

```bash
# Docker
sudo find /etc/apt/sources.list.d/ -name "*docker*" -exec rm -f {} \;
sudo rm -f /usr/share/keyrings/docker-archive-keyring.gpg

# Terraform
sudo find /etc/apt/sources.list.d/ -name "*hashicorp*" -exec rm -f {} \;
sudo rm -f /usr/share/keyrings/hashicorp-archive-keyring.gpg

# Kubectl
sudo find /etc/apt/sources.list.d/ -name "*kubernetes*" -exec rm -f {} \;
sudo rm -f /usr/share/keyrings/kubernetes-archive-keyring.gpg

# Mise à jour
sudo apt update
```

## Lancement du playbook Ansible

```bash
ansible-playbook -i "localhost," -c local setup_wsl.yml --ask-become-pass
```

## Outils installés

Le playbook installe et configure les outils suivants :

### Outils DevOps
- **Docker** : Plateforme de conteneurisation
- **Kubectl** : CLI Kubernetes
- **Terraform** : Infrastructure as Code

### Outils de développement
- **Vim** : Éditeur de texte avec Vundle
- **Tmux** : Multiplexeur de terminal avec plugins
- **Zsh** : Shell avancé avec Oh My Zsh et Powerlevel10k

### Plugins Zsh installés
- zsh-autosuggestions
- fzf-tab
- autojump
- git-open
- jsontools (intégré à Oh My Zsh)

### Plugins Tmux installés
- TPM (Tmux Plugin Manager)
- dracula-tmux
- nord-tmux
- tmux-resurrect
- tmux-continuum
- tmux-sensible

## Post-installation

### 1. Recharger le shell

```bash
exec zsh
```

### 2. Installer les plugins Tmux

Lancez tmux et appuyez sur `Prefix + I` (par défaut `Ctrl+b` puis `I`) pour installer les plugins.

### 3. Vérifier Docker

```bash
docker --version
docker ps
```

**Note** : Vous devrez peut-être vous déconnecter/reconnecter pour que le groupe Docker soit pris en compte.

### 4. Vérifier les autres outils

```bash
kubectl version --client
terraform version
vim --version
tmux -V
zsh --version
```

## Structure du projet

```
.
├── playbook.yml
├── inventory.yml
├── roles/
│   ├── docker/
│   │   ├── tasks/
│   │   │   └── main.yml
│   │   └── files/
│   │       └── config.json
│   ├── kubectl/
│   │   └── tasks/
│   │       └── main.yml
│   ├── terraform/
│   │   └── tasks/
│   │       └── main.yml
│   ├── vim/
│   │   ├── tasks/
│   │   │   └── main.yml
│   │   └── files/
│   │       └── .vimrc
│   ├── tmux/
│   │   ├── tasks/
│   │   │   └── main.yml
│   │   └── files/
│   │       └── .tmux.conf
│   └── zsh/
│       ├── tasks/
│       │   └── main.yml
│       └── files/
│           ├── .zshrc
│           └── .p10k.zsh
└── README.md
```

## Dépannage

### Erreur de conflit de repository

Si vous obtenez une erreur de type "Conflicting values set for option Signed-By", exécutez le nettoyage mentionné ci-dessus.

### Docker ne fonctionne pas

```bash
# Vérifier le service
sudo service docker status

# Démarrer Docker
sudo service docker start

# Ajouter votre utilisateur au groupe docker (si pas fait automatiquement)
sudo usermod -aG docker $USER
```

Déconnectez-vous et reconnectez-vous pour appliquer les changements.

### Zsh n'est pas le shell par défaut

```bash
chsh -s $(which zsh)
```

Fermez et rouvrez votre terminal.

### Problèmes de connectivité réseau avec VPN

Si vous êtes connecté à un VPN d'entreprise et que WSL n'a plus accès à internet :

#### Vérifier si c'est un problème DNS uniquement

```bash
# Tester avec une IP publique
ping 8.8.8.8

# Si ça fonctionne, c'est juste le DNS
```

Si c'est un problème DNS uniquement, créez ou éditez `/etc/wsl.conf` et ajoutez une configuration DNS personnalisée.

#### Installer wsl-vpnkit

Si le ping vers une IP publique ne fonctionne pas, wsl-vpnkit est nécessaire pour fournir la connectivité réseau à WSL 2 lorsqu'elle est bloquée par le VPN. Suivez les instructions dans la section "Installation de wsl-vpnkit" ci-dessus.

#### Vérifier le statut de wsl-vpnkit

```powershell
# Depuis PowerShell
wsl.exe -d wsl-vpnkit --cd /app service wsl-vpnkit status
```

Si le service n'est pas en cours d'exécution :

```powershell
wsl.exe -d wsl-vpnkit --cd /app service wsl-vpnkit start
```

#### Problème d'exécution de wsl-gvproxy.exe

Si vous obtenez l'erreur "wsl-gvproxy.exe is not executable", vérifiez que l'interopérabilité WSL est activée et que les permissions Windows sont correctes.

## Personnalisation

### Modifier les thèmes Tmux

Éditez `~/.tmux.conf` et changez la ligne du thème :

```bash
# Pour Dracula
set -g @plugin 'dracula/tmux'

# Pour Nord
set -g @plugin 'arcticicestudio/nord-tmux'
```

### Configurer Powerlevel10k

```bash
p10k configure
```

## Ressources

- [Documentation WSL](https://docs.microsoft.com/fr-fr/windows/wsl/)
- [wsl-vpnkit GitHub](https://github.com/sakai135/wsl-vpnkit)
- [Documentation Docker](https://docs.docker.com/)
- [Documentation Kubernetes](https://kubernetes.io/docs/)
- [Documentation Terraform](https://www.terraform.io/docs/)
- [Oh My Zsh](https://ohmyz.sh/)
- [Powerlevel10k](https://github.com/romkatv/powerlevel10k)
- [Tmux](https://github.com/tmux/tmux/wiki)

## Licence
