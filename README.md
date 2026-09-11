# Équilibrage de charge avec HAProxy — Grappe de 2 serveurs web

Projet tutoré réalisé dans le cadre de ma formation en Réseaux Informatiques et Télécommunications à l'ISGE-BF (2023–2024), en binôme avec Rachid Sankara, sous la supervision de M. Alexis Nagalo.

## 🎯 Objectif

Mettre en place une architecture haute disponibilité en répartissant la charge entre deux serveurs web (Apache) grâce à un load balancer HAProxy, afin de garantir la disponibilité et la performance d'un service web même en cas de forte affluence ou de panne d'un des serveurs.

## 🏗️ Architecture

```
                    ┌─────────────────┐
   Clients ────────▶│  HAProxy (LB)    │
                    │ 192.168.72.157   │
                    └────────┬─────────┘
                             │ round robin
               ┌─────────────┴─────────────┐
               ▼                           ▼
    ┌──────────────────┐        ┌──────────────────┐
    │   WebServer 1     │        │   WebServer 2     │
    │ 192.168.72.158    │        │ 192.168.72.159    │
    │   Apache2         │        │   Apache2          │
    └──────────────────┘        └──────────────────┘
```

## 🛠️ Environnement & outils

- **Système** : Ubuntu Server (Linux)
- **Virtualisation** : VMware
- **Serveur web** : Apache2
- **Load balancer** : HAProxy (algorithme round robin)

## ⚙️ Étapes de mise en œuvre

### 1. Installation d'Apache sur chaque serveur web
```bash
sudo apt-get install apache2
sudo systemctl enable apache2
sudo systemctl start apache2
sudo ufw allow 80/tcp
```

Une page d'accueil personnalisée a été créée sur chaque nœud pour identifier facilement quel serveur répond à la requête :
```bash
echo "<H1>Hello! This is webserver1: 192.168.72.158</H1>" | sudo tee /var/www/html/index.html
```

### 2. Installation de HAProxy sur le load balancer
```bash
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install haproxy
```

### 3. Configuration de HAProxy
Fichier `/etc/haproxy/haproxy.cfg` :
```
frontend web-frontend
    bind 192.168.72.157:80
    mode http
    default_backend web-backend

backend web-backend
    balance roundrobin
    server web-server1 192.168.72.158:80 check
    server web-server2 192.168.72.159:80 check
```

Démarrage du service :
```bash
sudo systemctl start haproxy
sudo systemctl enable haproxy
sudo systemctl status haproxy
```

## ✅ Résultat

En accédant à l'adresse virtuelle du load balancer (`192.168.72.157`), les requêtes sont automatiquement réparties entre les deux serveurs selon l'algorithme round robin — chaque rafraîchissement de page peut faire répondre un serveur différent, de façon totalement transparente pour le client.

## 📚 Ce que j'ai appris

- Les principes de la haute disponibilité et de la tolérance aux pannes (failover / failback)
- La configuration concrète d'un reverse proxy / load balancer avec HAProxy
- La gestion de services réseau sous Linux (systemctl, pare-feu, ports)
- Le fonctionnement des architectures en cluster (actif/actif, actif/passif)

## 📄 Rapport complet

Le rapport détaillé du projet (cahier des charges, état de l'art, bilan) est disponible dans ce dépôt.
