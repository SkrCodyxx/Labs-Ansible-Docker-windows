# **Labs-Ansible-Docker-Windows**

Ce document fournit un guide complet pour configurer un environnement Ansible avec une machine Ubuntu comme maître (control node) et une machine Windows comme worker (managed node). Ce projet utilise la machine Windows disponible sur [dockur/windows](https://github.com/dockur/windows/tree/master?tab=readme-ov-file). Merci à l'équipe de [dockur](https://github.com/dockur/windows) pour leur travail exceptionnel qui a permis d'utiliser cette machine.

---

# **Labs-Ansible-Docker-Windows**

This document provides a comprehensive guide to setting up an Ansible environment with an Ubuntu machine as the control node and a Windows machine as the managed node. This project uses the Windows machine available from [dockur/windows](https://github.com/dockur/windows/tree/master?tab=readme-ov-file). Special thanks to the [dockur](https://github.com/dockur/windows) team for their outstanding work that made this machine available.

---

## **Table des matières / Table of Contents**

1. [Introduction](#introduction)
2. [Prérequis / Prerequisites](#prérequis--prerequisites)
3. [Étape 1 : Préparation de la machine maître Ubuntu / Step 1: Preparing the Ubuntu Master Machine](#étape-1--préparation-de-la-machine-maître-ubuntu--step-1-preparing-the-ubuntu-master-machine)
4. [Étape 2 : Configuration de la machine Windows pour Ansible / Step 2: Configuring the Windows Machine for Ansible](#étape-2--configuration-de-la-machine-windows-pour-ansible--step-2-configuring-the-windows-machine-for-ansible)
5. [Étape 3 : Configurer le fichier `hosts` d’Ansible / Step 3: Configuring the Ansible `hosts` File](#étape-3--configurer-le-fichier-hosts-dansible--step-3-configuring-the-ansible-hosts-file)
6. [Étape 4 : Installer les modules nécessaires pour Windows / Step 4: Installing Required Modules for Windows](#étape-4--installer-les-modules-nécessaires-pour-windows--step-4-installing-required-modules-for-windows)
7. [Étape 5 : Exemple de tâches avec Ansible / Step 5: Task Examples with Ansible](#étape-5--exemple-de-tâches-avec-ansible--step-5-task-examples-with-ansible)
8. [Conclusion](#conclusion)

---

## **Introduction**

Ce projet montre comment gérer une machine Windows via Ansible en utilisant Docker pour exécuter le nœud maître Ubuntu. Il inclut des exemples pour tester la connectivité et exécuter des tâches sur le worker Windows.

---

This project demonstrates how to manage a Windows machine using Ansible with Docker running the Ubuntu control node. It includes examples to test connectivity and perform tasks on the Windows worker.

---

## **Prérequis / Prerequisites**

- Docker installé sur l’hôte pour créer un conteneur Ubuntu.

### **1. Système d'exploitation**
Assurez-vous que vous utilisez Ubuntu ou une autre distribution Linux compatible.

### **2. Installation de Docker**
Pour installer Docker, exécutez les commandes suivantes :
```bash
sudo apt update
sudo apt install -y docker.io
```

### **3. Installation de Docker Compose**
Pour installer Docker Compose, utilisez la commande suivante :
```bash
sudo apt install -y docker-compose
```

### **4. Installation de Python**
Ansible nécessite Python. Assurez-vous que Python est installé :
```bash
sudo apt install -y python3 python3-pip
```

### **5. Installation des paquets nécessaires pour KVM**
Pour utiliser KVM sur votre machine, exécutez les commandes suivantes pour l'installation et la configuration :
```bash
sudo apt update
sudo apt install -y qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils
```

### **6. Vérification de l'installation de KVM**
Pour vérifier que votre système prend en charge KVM, exécutez la commande suivante :
```bash
sudo kvm-ok
```

### **7. Démarrage et vérification du service libvirt**
Assurez-vous que le service `libvirtd` est actif et démarré :
```bash
sudo systemctl status libvirtd
sudo systemctl start libvirtd
```

### **8. Vérification de la présence de KVM**
Pour confirmer que le périphérique KVM est présent, exécutez la commande suivante :
```bash
ls /dev/kvm
```

---

## **0. Création du répertoire de travail et configuration initiale / Work Directory Creation and Initial Setup**

### 0.1 Créer un répertoire de travail / Create a Work Directory
Before starting, create a work directory on your Ubuntu machine to organize the files and configurations needed for your project. You can do this using the following command:
```bash
mkdir ansible-windows-setup
cd ansible-windows-setup
```

### 0.2 Créer le fichier `docker-compose.yml` / Create the `docker-compose.yml` File
Create a `docker-compose.yml` file in your work directory using the following command:
```bash
touch docker-compose.yml
```

### 0.3 Copier le contenu de la configuration Docker Compose / Copy the Docker Compose Configuration Content
To set up the Docker container with Ansible and WinRM, copy the content from the following file into your `docker-compose.yml`:

[Content of the Docker Compose file](https://gitlab.com/Codyxxx/labs-ansible-docker-windows/-/blob/main/Docker_compose_file/docker-compose.yml?ref_type=heads)

You can use `curl` or `wget` to fetch the file directly into your work directory as shown below:
```bash
curl -o docker-compose.yml https://gitlab.com/Codyxxx/labs-ansible-docker-windows/-/raw/main/Docker_compose_file/docker-compose.yml
```

### 0.4 Enregistrer et lancer le conteneur / Save and Start the Container
Enregistrez le fichier et lancez / save the file and start:
```bash
docker-compose up -d
```

---
If you need help customizing your `docker-compose.yml` file or have any other questions, feel free to ask!

## **Étape 1 : Préparation de la machine maître Ubuntu / Step 1: Preparing the Ubuntu Master Machine**

### 1.1 Se connecter au conteneur Ubuntu maître / Connect to the Ubuntu Master Container
```bash
docker exec -it ubuntu_master bash
```

### 1.2 Mettre à jour les paquets / Update Packages
```bash
apt update && apt upgrade -y
```

### 1.3 Installer Ansible / Install Ansible
1. Ajouter le dépôt officiel Ansible / Add the official Ansible repository:
   ```bash
   apt install -y software-properties-common
   add-apt-repository --yes --update ppa:ansible/ansible
   ```
2. Installer Ansible / Install Ansible:
   ```bash
   apt install -y ansible
   ```
3. Vérifier l’installation / Verify installation:
   ```bash
   ansible --version
   ```

---

## **Étape 2 : Configuration de la machine Windows pour Ansible / Step 2: Configuring the Windows Machine for Ansible**

### 2.1 Accéder à la machine Windows / Access the Windows Machine
**Choix 1 / Option 1**: Access http://localhost:8006 via your browser (NoVNC).

**Choix 2 / Option 2**: Use an RDP application like Remmina to connect.

### 2.2 Configurer WinRM sur Windows / Configure WinRM on Windows
1. Exécutez PowerShell en tant qu'administrateur / Run PowerShell as Administrator.
2. Configurez WinRM / Configure WinRM:
   ```powershell
   winrm quickconfig
   winrm quickconfig -q
   winrm set winrm/config/service/auth '@{Basic="true"}'
   winrm set winrm/config/service '@{AllowUnencrypted="true"}'
   ```

### 2.3 Vérifier la configuration WinRM / Verify WinRM Configuration
1. Vérifiez que le port **5985** est ouvert / Ensure port **5985** is open:
   ```powershell
   Test-NetConnection -ComputerName localhost -Port 5985
   ```
2. Vérifiez les paramètres / Verify settings:
   ```powershell
   winrm enumerate winrm/config/Listener
   ```

---

## **Étape 3 : Configurer le fichier `hosts` d’Ansible / Step 3: Configuring the Ansible `hosts` File**

### 3.1 Éditer le fichier d’inventaire / Edit the Inventory File
1. Retournez sur la machine maître Ubuntu / Return to the Ubuntu master machine:
   ```bash
   cd
   ```
2. Créez un fichier `inventory.ini` / Create an `inventory.ini` file:
   ```bash
   nano inventory.ini
   ```
3. Ajoutez les détails de la machine Windows / Add the Windows machine details:
   ```ini
   [windows]
   winS2019 ansible_host=192.168.2.10

   [windows:vars]
   ansible_user=administrator
   ansible_password=abc.123
   ansible_connection=winrm
   ansible_winrm_transport=basic
   ```

### 3.2 Tester la connectivité / Test Connectivity
Exécutez la commande suivante / Run the following command:
```bash
ansible all -m ansible.windows.win_ping -i inventory.ini
```

---

## **Étape 4 : Installer les modules nécessaires pour Windows / Step 4: Installing Required Modules for Windows**

### 4.1 Installer les collections Ansible pour Windows / Install Ansible Collections for Windows
```bash
ansible-galaxy collection install ansible.windows
ansible-galaxy collection install community.windows
```

### 4.2 Tester les modules Windows / Test Windows Modules
1. Créez un fichier de test `test-windows.yml` / Create a `test-windows.yml` file:
   ```yaml
   ---
   - name: Test connection to Windows
     hosts: windows
     tasks:
       - name: Ping Windows machine
         ansible.windows.win_ping:
   ```
2. Exécutez le fichier de test / Run the test file:
   ```bash
   ansible-playbook -i inventory.ini test-windows.yml
   ```

---

## **Étape 5 : Exemple de tâches avec Ansible / Step 5: Task Examples with Ansible**

### 5.1 Installer une application sur Windows / Install an Application on Windows
1. Créez un fichier de tâche `install-app.yml` / Create a task file `install-app.yml`:
   ```yaml
   ---
   - name: Install Notepad++ on Windows
     hosts: windows
     tasks:
       - name: Download Notepad++ installer
         win_get_url:
           url: https://github.com/notepad-plus-plus/notepad-plus-plus/releases/download/v8.5.6/npp.8.5.6.Installer.exe
           dest: C:\\Temp\\npp_installer.exe

       - name: Install Notepad++
         win_package:
           path: C:\\Temp\\npp_installer.exe
           arguments: /S
           state: present
   ```
2. Exécutez la tâche / Run the task:
  



### 5.2 Configurer des services Windows / Configure Windows Services
- Creer `manage-service.yml` / Example `manage-service.yml` playbook:

**Configurer des services Windows** :
   - Exemple pour démarrer un service :
     ```yaml
     ---
     - name: Manage Windows services
       hosts: windows
       tasks:
         - name: Ensure a service is running
           ansible.windows.win_service:
             name: wuauserv
             state: started
     ```
   - Exécutez le playbook :
     ```bash
     ansible-playbook manage-service.yml -i inventory.ini
     ```

---

### Conclusion

Ce document a couvert l'installation et la configuration d'Ansible pour gérer une machine Windows, en utilisant Docker comme environnement pour le nôde maître Ubuntu. Nous avons examiné les étapes de connexion, la configuration de WinRM, la modification du fichier d'inventaire d'Ansible et les exemples de tâches pour exécuter des commandes et scripts sur le nôde Windows. Vous pouvez utiliser ce guide comme point de départ pour d'autres projets d'automatisation avec Ansible sur Windows.

Si vous avez besoin d'aide pour des étapes spécifiques ou pour personnaliser vos tâches, n'hésitez pas à me le faire savoir !