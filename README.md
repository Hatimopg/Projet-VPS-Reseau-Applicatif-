# 🚀 VPS Security & Cybersecurity Project

## 📌 Overview

Ce projet présente la mise en place complète d’un **serveur VPS sécurisé**, avec une approche orientée **cybersécurité offensive et défensive**.

L’objectif était de simuler un environnement réel exposé à Internet, puis de :
- sécuriser le serveur
- détecter les attaques
- analyser les comportements malveillants

---

## 🧠 Objectifs

- Hardening d’un serveur Linux
- Protection contre les attaques brute-force
- Mise en place d’un honeypot
- Monitoring en temps réel
- Configuration d’un serveur web + mail

---

## 🛠️ Stack Technique

- Linux (Ubuntu)
- SSH (clé + port personnalisé)
- UFW (Firewall)
- Fail2Ban
- Nginx (Web Server)
- Cloudflare (DNS & protection)
- Cowrie (Honeypot SSH)
- Netdata (Monitoring)
- Grafana
- Postfix / Mailutils
- Docker (tests avec Mailcow)

---

## 🔐 Sécurisation SSH

- Port modifié (22 → 1949)
- Accès root désactivé
- Authentification par clé uniquement

```bash
sudo nano /etc/ssh/sshd_config.d/50-cloud-init.conf
