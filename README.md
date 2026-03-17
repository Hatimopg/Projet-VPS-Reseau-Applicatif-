# 🛡️ VPS Hardening, Threat Intel & Mail Stack (Project: Brocoli / dontbesad.tech)

Projet de sécurisation offensive et défensive d'une infrastructure cloud. L'objectif est simple : **Hardening total**, **Zero Trust** et **Analyse active des attaquants**. Ce dépôt contient mes configurations réelles et mes méthodologies de déploiement.

---

### 🔓 1. SSH Hardening (Anti-Bruteforce) 
Passage d'une configuration standard vulnérable à un accès restreint et furtif.
* **Obscurité :** Migration du port `22` vers le port `1949` (mon favori).
* **IAM :** Suppression de l'accès `root`, création d'un user dédié et déploiement de clés SSH obligatoires.
* **Config :** Modification via `sudo nano /etc/ssh/sshd_config.d/50-cloud-init.conf`.
* **Result :** 99% des bots de scan automatisés échouent avant même la tentative d'auth.

### 🧱 2. Firewalling & Segmentation (UFW)
Politique de whitelist stricte pour verrouiller le périmètre.
* **Whitelist IP :** Seule mon IP publique est autorisée sur le port `1949`. Tout le reste est **DROP**.
* **IPv6 Killing :** Désactivation de l'IPv6 dans `/etc/default/ufw` pour supprimer une surface d'attaque souvent oubliée.
* **Monitoring :** `sudo ufw status verbose` pour valider l'étanchéité du flux.

### ⛓️ 3. Intrusion Prevention System (Fail2Ban)
Mise en place de "prisons" (jails) dynamiques pour bannir les persistants.
* **Jail [sshd] :** Port `1949`, `maxretry = 3`, `findtime = 600`, `bantime = 3600`.
* **Jail [sshd-preauth] :** Filtre personnalisé (`/etc/fail2ban/filter.d/sshd-preauth.conf`) pour détecter les fermetures de connexion suspectes durant la phase d'auth.
* **Manual Ban :** Utilisation de `sudo fail2ban-client set sshd-preauth banip [IP]` pour une réponse immédiate aux incidents.

### 🍯 4. Active Deception: Cowrie Honeypot
Parce que la défense ne suffit pas, j'observe l'attaquant dans son habitat naturel.
* **Leurre SSH :** Déploiement de **Cowrie** sur le port `2222`.
* **Logs & Analyse :** Script `analyse_logs.py` pour parser `cowrie.json` et extraire les IPs/Payloads.
* **Prison Cowrie :** `maxretry = 2`, `bantime = 86000` (ban de 24h automatique pour tout attaquant touchant au honeypot).
* **Banner Custom :** Configuration de `banner.txt` pour simuler un environnement vulnérable crédible.

### 🌐 5. Web Stack & Reverse Proxy (Nginx + Cloudflare)
Déploiement sécurisé du domaine `dontbesad.tech`.
* **TLS Hardening :** Uniquement `TLSv1.2` et `TLSv1.3`. Ciphers `HIGH:!aNULL:!MD5`.
* **Headers de Sécurité :** `X-Frame-Options DENY` (Anti-Clickjacking) et `X-Content-Type-Options nosniff`.
* **Edge Security :** Proxy Cloudflare avec enregistrements `A` et certificats d'origine stockés localement.
* **AI Integration :** Chatbot OpenAI payé 5€ (ma3lich) avec backend PHP (php-fpm) pour l'interaction utilisateur.

### 📧 6. Mail Infrastructure (Postfix & Mailcow Dockerized)
Gestion souveraine des mails avec validation cryptographique.
* **Stack Docker :** Déploiement de Mailcow (Postfix, Dovecot, Rspamd) via `docker-compose`.
* **Délivrabilité :** Configuration DKIM (`opendkim`), SigningTable et TrustedHosts pour éviter les spams.
* **Tests :** Utilisation de `mailutils` et `mutt` pour valider l'envoi/réception (`echo "Test" | mail -s "Subject" ...`).
* **Alias de Secours :** `postmaster` et `admin` redirigés vers mon user principal pour les alertes critiques.

### 📊 7. Monitoring & Stealth Ops (Netdata)
Monitoring temps réel sans exposition publique.
* **Netdata :** Installation via script kickstart sur le port `19999`.
* **Tunneling SSH :** Accès aux metrics via `ssh -p 1949 -L 19999:localhost:19999`. Le port dashboard reste fermé au public (127.0.0.1 only).

### 🧪 8. Experimental & Lab
* **Socat Deception :** Simulation d'un faux serveur sur le port `25565` avec message de troll pour les scanners : `"§cC'est un faux serveur mdr t genant"`.
* **Wireguard :** (WIP) Test d'implémentation VPN pour accès backend.

---
**⚡ Conclusion :** Ce serveur n'est pas une simple machine, c'est un laboratoire de sécurité actif. Chaque tentative de connexion est loggée, chaque IP suspecte est bannie, et chaque service est durci au maximum.
