# 🚀 Projet : Déploiement Kubernetes K3d avec Packer & Ansible

## 📋 Description
Ce projet démontre le déploiement automatisé d'une application Nginx customisée sur un cluster Kubernetes K3d, en utilisant Packer et Ansible.

## 🏗️ Architecture
index.html (SVG) -> Packer -> Image Docker -> K3d -> Ansible -> Kubernetes

## ✅ Prérequis
- GitHub Codespace
- K3d (1 master + 2 workers)
- kubectl

## 🔧 Installation

### 1. Packer et Ansible
```bash
sudo apt update
sudo apt install packer ansible -y
