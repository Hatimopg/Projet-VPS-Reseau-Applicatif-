# 🛡️ VPS Hardening, Threat Intel & Mail Stack (Project: Brocoli / dontbesad.tech)

> ⚠️ **DISCLAIMER :** Ce serveur et cette infrastructure n'existent plus à l'heure actuelle. Toutes les adresses IP, clés API, domaines et configurations sensibles mentionnés ici sont obsolètes et ne présentent plus aucun risque de sécurité. Ce dépôt est une archive technique à but de démonstration.

---

### 🔓 1. SSH Hardening (Anti-Bruteforce) 
Passage d'une configuration standard vulnérable à un accès furtif et durci.
* **Obscurité :** Migration du port `22` vers le port `1949` (mon favori).
* **IAM :** Suppression de l'accès `root`, création d'un user dédié et authentification par clés SSH uniquement.
* **Config :** Hardening via `sudo nano /etc/ssh/sshd_config.d/50-cloud-init.conf`.
* **Result :** 99% des bots de scan automatisés échouent avant même la tentative d'auth.

### 🧱 2. Firewalling & Segmentation (UFW)
Politique de whitelist stricte pour verrouiller le périmètre réseau.
* **Whitelist IP :** Seule mon adresse IP publique est autorisée sur le port `1949` via UFW. Tout le reste est **DROP**.
* **IPv6 Killing :** Désactivation de l'IPv6 dans `/etc/default/ufw` pour supprimer une surface d'attaque invisible.
* **Vérification :** `sudo ufw status verbose` (UFW actif et restrictif).

### ⛓️ 3. Intrusion Prevention System (Fail2Ban)
Mise en place de "prisons" (jails) dynamiques pour bannir les attaquants persistants.
* **Jail [sshd] :** Port `1949`, `maxretry = 3`, `findtime = 600`, `bantime = 3600`.
* **Jail [sshd-preauth] :** Filtre custom (`/etc/fail2ban/filter.d/sshd-preauth.conf`) avec regex pour intercepter les fermetures de connexion suspectes : `failregex = ^.sshd.Connection closed by authenticating user .* port \d+ \[preauth\]$`.
* **Manual Ban :** Utilisation de `sudo fail2ban-client set sshd-preauth banip 14.103.112.106`.

### 🍯 4. Active Deception: Cowrie Honeypot
Détournement des attaquants vers un environnement contrôlé pour analyse.
* **Leurre SSH :** Déploiement de **Cowrie** sur le port `2222`.
* **Persistence :** Création d'une prison Fail2Ban dédiée (`/etc/fail2ban/jail.d/cowrie.local`) : `maxretry = 2`, `bantime = 86000`.
* **Analyse :** Script `analyse_logs.py` pour parser les tentatives et génération d'un tableau d'IPs attaquantes via `sudo python3 analyse_logs.py`.
* **Bannière :** Customisation du `banner.txt` et de la base d'utilisateurs `userdb.txt`.

### 🌐 5. Web Stack & Reverse Proxy (Nginx + Cloudflare)
Déploiement sécurisé du domaine `dontbesad.tech`.
* **VHost Hardening :** Redirection 301 systématique vers HTTPS.
* **TLS & Headers :** Uniquement `TLSv1.2/1.3`, ciphers `HIGH:!aNULL:!MD5`. Ajout de `X-Frame-Options DENY` et `X-Content-Type-Options nosniff`.
* **AI Integration :** Intégration d'un chatbot via API OpenAI (payée 5€ ma3lich) avec backend PHP (`php-fpm`).
* **Cloudflare :** Utilisation des DNS Cloudflare (Type A) et certificats d'origine stockés localement dans Nginx.

### 📧 6. Mail Infrastructure (Postfix & Mailcow Dockerized)
Gestion souveraine des flux mails avec authentification cryptographique.
* **Docker Stack :** Installation de Docker & Docker Compose pour faire tourner **Mailcow**.
* **Auth & DKIM :** Configuration de `opendkim` (KeyTable, SigningTable, TrustedHosts) pour signer les mails.
* **Postfix Config :** Modification du `main.cf` pour intégrer les milters et les alias (`sudo newaliases`).
* **Test Ops :** Tests d'envoi via `mailutils` et gestion de l'affichage via `~/.muttrc` (affichage du nom Admin).

### 📊 7. Monitoring & Stealth Ops (Netdata)
Monitoring temps réel sans exposition de surface.
* **Netdata :** Installation via kickstart script. Port `19999` ouvert uniquement en local.
* **Tunneling SSH :** Accès sécurisé aux statistiques via `ssh -p 1949 -L 19999:localhost:19999 darkprince667@179.61.190.57`.

### 🧪 8. Experimental & Socat
* **Socat Troll :** Simulation d'un faux service sur le port `25565` avec un message d'accueil personnalisé pour les curieux : `"§cC'est un faux serveur mdr t genant"`.

---

**⚡ Conclusion & Feedback :** Ce projet m'a pris **3 mois** de travail intensif, entre les phases de configuration, de debug et de sécurisation. Bien que l'infrastructure soit aujourd'hui hors ligne, le **gain d'expérience** en administration système, en hardening Linux et en gestion de stack Docker était le but principal. Chaque tentative d'intrusion détectée par mes honeypots a été une leçon de cybersécurité appliquée.
