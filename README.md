# 🚀 Projet : Déploiement Kubernetes K3d avec Packer & Ansible

## 📋 Description
Ce projet démontre le déploiement automatisé d'une application Nginx customisée (dessin SVG) sur un cluster Kubernetes K3d dans GitHub Codespace.

**Technologies utilisées :**
- **K3d** : Cluster Kubernetes léger (1 master + 2 workers)
- **Packer** : Construction de l'image Docker personnalisée
- **Ansible** : Automatisation du déploiement sur Kubernetes
- **GitHub Codespace** : Environnement de développement cloud

## 🏗️ Architecture


## 📊 Applications déployées

| Application | Port | Accès |
|-------------|------|-------|
| Mario Game | 8080 | https://[workspace]-8080.github.dev |
| Custom Nginx (SVG) | 8081 | https://[workspace]-8081.github.dev |

---

## 🔧 Séquence 1 & 2 : Installation du cluster K3d

### 1. Création du cluster
```bash
curl -s https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash
k3d cluster create lab --servers 1 --agents 2
kubectl get nodes


2. Déploiement de Mario (test)

kubectl create deployment mario --image=sevenajay/mario
kubectl expose deployment mario --type=NodePort --port=80
kubectl port-forward svc/mario 8080:80 >/tmp/mario.log 2>&1 &

🐳 Séquence 3 : Déploiement personnalisé
3.1 Installation Packer & Ansible
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update
sudo apt install packer ansible -y

3.2 Installation du plugin Docker Packer

cat > packer-plugin.pkr.hcl << 'PLUGIN'
packer {
  required_plugins {
    docker = {
      version = ">= 1.0.0"
      source  = "github.com/hashicorp/docker"
    }
  }
}
PLUGIN
packer init packer-plugin.pkr.hcl

3.3 Création du template Packer

cat > packer-template.json << 'JSON'
{
  "builders": [{
    "type": "docker",
    "image": "nginx:alpine",
    "commit": true
  }],
  "provisioners": [{
    "type": "file",
    "source": "index.html",
    "destination": "/usr/share/nginx/html/index.html"
  }],
  "post-processors": [{
    "type": "docker-tag",
    "repository": "custom-nginx",
    "tag": "latest"
  }]
}
JSON

3.4 Build de l'image avec Packer

packer build packer-template.json
docker images | grep custom-nginx

3.5 Import dans K3d

k3d image import custom-nginx:latest -c lab

3.6 Déploiement avec Ansible

pip3 install kubernetes --user
ansible-galaxy collection install kubernetes.core
ansible-playbook deploy.yml


3.7 Port forwarding

kubectl port-forward svc/custom-nginx 8081:80 >/tmp/custom-nginx.log 2>&1 &
curl -s -o /dev/null -w "HTTP: %{http_code}\n" http://localhost:8081

Vérification
kubectl get pods
kubectl get svc


Sortie attendue
NAME                           READY   STATUS    RESTARTS   AGE
custom-nginx-d9c688597-qhgvd   1/1     Running   0          2m
custom-nginx-d9c688597-v7h7v   1/1     Running   0          2m
mario-64494f9d-9nnnk           1/1     Running   0          43m

NAME           TYPE        CLUSTER-IP    PORT(S)        AGE
custom-nginx   NodePort    10.43.22.88   80:30080/TCP   2m
mario          NodePort    10.43.77.25   80:32582/TCP   44m