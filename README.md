# 🚀 Exercice Global – CI/CD avec Ansible, Docker & GitHub Actions

[![CI](https://github.com/Krynclv/aws-exo5/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/Krynclv/aws-exo5/actions/workflows/ci.yml)

## 📌 Objectif
Mettre en place une chaîne complète d’intégration et déploiement continu (CI/CD) :
- Code versionné avec GitHub (branches main, dev, feature/**)
- Tests automatisés avec **pytest**
- Environnements **serveur-dev** et **serveur-prod** sous Docker
- Déploiement automatisé avec **Ansible** via un **runner self-hosted**
- Sécurité avec gestion des secrets SSH & sudo dans GitHub

---

## 🛠️ Étapes réalisées

### 1. Initialisation
- Création du repository GitHub
- Création des premiers fichiers Python (`app.py`, `test_app.py`)
- Ajout d’un `.gitignore` et `requirements.txt`
- Mise en place d’un premier workflow GitHub Actions pour exécuter les tests

### 2. Docker & Ansible
- Écriture du `docker-compose.yml` pour lancer deux conteneurs SSH :
  - `serveur-dev` (port 2222)
  - `serveur-prod` (port 2200)
- Connexion SSH fonctionnelle depuis Ansible
- Création de `inventory.ini` pour décrire les hôtes

### 3. Playbooks Ansible
- `deploy-dev.yml` → déploie la branche **dev/feature** sur `serveur-dev`
- `deploy-prod.yml` → déploie la branche **main** sur `serveur-prod`
- Chaque playbook :
  - installe les dépendances (python3, git, node, nginx, curl…)
  - clone le repo GitHub
  - exécute un build si projet Node.js
  - dépose une page de confirmation (`/var/www/html/index.html`)
  - écrit un log de déploiement (`/var/log/deploy.log`)
  - démarre nginx si nécessaire

### 4. GitHub Actions (CI/CD)
- Workflow `.github/workflows/ci.yml`
  - **tests** : exécution des tests Python (pytest) sur `ubuntu-latest`
  - **deploy** : exécution des playbooks Ansible via un **runner self-hosted**
    - push sur `dev` ou `feature/*` → déploiement **serveur-dev**
    - push sur `main` → déploiement **serveur-prod**

### 5. Runner self-hosted
- Installation et configuration du runner GitHub sur Kali
- Vérification qu’il est **online** et exécute les jobs de déploiement

### 6. Sécurité & Secrets
- Génération d’une paire SSH (ed25519)
- Secrets GitHub configurés :
  - `DEV_SSH_KEY` / `PROD_SSH_KEY` → clé privée
  - `DEV_SUDO_PASS` / `PROD_SUDO_PASS` → mot de passe sudo
- Clé publique copiée dans `~/.ssh/authorized_keys` sur les serveurs
- Connexion SSH sans mot de passe validée

---

## ✅ Déclenchement & Vérification

### Déploiement DEV
```bash
git checkout dev
echo "- update $(date)" >> README.md
git add README.md
git commit -m "test: trigger dev deploy"
git push origin dev
