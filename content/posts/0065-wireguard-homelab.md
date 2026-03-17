---
title: "Homelab: exposer ses services homelab avec WireGuard, Traefik et Let's Encrypt"
summary: "Homelab: exposer ses services homelab avec WireGuard, Traefik et Let's Encrypt"
date: 2026-03-17T00:00:00+01:00
draft: false
tags: ['homelab', 'wireguard']
pin: false
---

# Homelab: exposer ses services homelab avec WireGuard, Traefik et Let's Encrypt

Dans mon homelab, je fais tourner plusieurs applications (comme Immich) sur un simple PC derrière une connexion FAI résidentielle avec IP dynamique. Je ne veux pas exposer mes services directement sur Internet, mais je souhaite quand même y accéder de manière sécurisée depuis mon téléphone ou en déplacement.

Ma solution repose sur un principe simple : **un seul port ouvert sur mon routeur, celui du VPN WireGuard**.

## Prérequis
- Un **nom de domaine**
- Un **registrar DNS avec API** (OVH dans mon cas, mais n'importe lequel peut fonctionner)
- Le routeur internet configuré pour autoriser le forwarding du traffic VPN. Une seule règle suffit :
    - **Protocole** : UDP
    - **Port externe** : 52420
    - **IP interne** : Votre serveur homelab
    - **Port interne** : 51820
- et **docker compose** pour faire simple

## Principe de base

**Depuis l'extérieur (via mobile):**

1. Client se connecte au VPN (vpn.myhomelab.tld:58237)
2. Client obtient une IP 10.13.13.X et utilise DNS 172.16.0.253
3. Client demande par exemple immich.myhomelab.tld
4. dnsdist résout vers l'IP locale de Traefik
5. Connexion HTTPS vers Traefik (certificat Let's Encrypt)
6. Traefik proxifie vers réseau interne docker (immich:2283)

```
Internet
   │
   │ Port UDP de WireGuard uniquement
   ▼
┌─────────────────────────────────────┐
│      Router (NAT)                   │
└─────────────────────────────────────┘
   │
   │ Réseau local 172.16.0.0/12
   ▼
┌──────────────────────────────────────────────────────────────────┐
│                    Homelab Server (172.16.0.0/12)                │
│                                                                  │
│  ┌────────────────┐  ┌────────────────────┐  ┌─────────────────┐ │
│  │ publicaddr     │  │   WireGuard        │  │    dnsdist      │ │
│  └────────────────┘  └────────────────────┘  └─────────────────┘ │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐    │
│  │                    Traefik (reverse proxy)               │    │
│  │                                                          │    │
│  │  • Écoute sur ports 80/443 (local uniquement)            │    │
│  │  • Génère certificats Let's Encrypt via DNS-01           │    │
│  │  • Auto-découverte via labels Docker                     │    │
│  └──────────────────────────────────────────────────────────┘    │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐     │
│  │              Services (Immich, Vault,etc...)            │     │
│  └─────────────────────────────────────────────────────────┘     │
└──────────────────────────────────────────────────────────────────┘
```

## Les composants clés

### WireGuard 

Dans mon setup, WireGuard :
- Crée un réseau privé 10.13.13.0/24 pour les clients
- Pousse le DNS 172.16.0.253 vers les clients
- Route uniquement le trafic vers 172.16.0.0/12 (split-tunneling)
- Maintient les connexions avec keepalive

**L'image Docker linuxserver/wireguard** génère automatiquement les configurations pour chaque peer (device).

### PublicADDR

Ne disposant pas d'un IP publique stable, j'utilise **publicaddr-ovhcloud** pour que `vpn.myhomelab.tld` pointe toujours vers la bonne adresse, 

Ce container :
- Détecte l'IP publique toutes les 5 minutes
- Compare avec l'IP enregistrée chez OVH
- Met à jour les enregistrements DNS si changement
- Supporte IPv4 et IPv6
- Gère plusieurs sous-domaines

C'est essentiellement un client DynDNS qui utilise l'API OVH.
> https://github.com/dmachard/docker-publicaddr-ovhcloud
> https://github.com/dmachard/python-publicaddr

### DnsDIST

Les clients VPN doivent résoudre `immich.monhomelab.tld` vers une IP **locale** (172.16.0.X) et non vers l'IP publique. C'est le principe du **split-brain DNS**.

**dnsdist (port 53)** :
- Point d'entrée pour toutes les requêtes DNS
- Filtrage de publicités via blocklists sur mobile 
- Logging vers dnscollector
- Forwarding vers divers DNS (google, quaddns, etc...)

**dnscollector** : https://github.com/dmachard/DNS-collector
- Collecte et agrège les logs DNS
- Statistiques et monitoring

### Traefik

Traefik sert de point d'entrée unique pour tous mes services web.

Son rôle :
- **Routage** : Dirige les requêtes vers le bon service selon le domaine
- **TLS termination** : Gère le HTTPS et les certificats
- **Auto-configuration** : Détecte les nouveaux services Docker via labels
- **ACME automatique** : Génère et renouvelle les certificats Let's Encrypt

**Utilisation du challenge DNS-01**

Le challenge HTTP-01 classique nécessite que Let's Encrypt puisse accéder à votre serveur sur le port 80 depuis Internet. Ça impliquerait d'exposer Traefik publiquement.

Le **challenge DNS-01** fonctionne différemment :
1. Traefik demande un certificat pour `immich.monhomelab.tld`
2. Let's Encrypt demande de prouver le contrôle du domaine
3. Traefik crée un enregistrement TXT via l'API OVH : `_acme-challenge.monhomelab.tld`
4. Let's Encrypt vérifie l'enregistrement DNS
5. Certificat émis !


## Mise en oeuvre avec docker

### Arborescence

homelab/
├── corelab/
│   ├── docker-compose.yml
│   └── .env
│
├── observe/
│   ├── docker-compose.yml
│   └── .env
│
├── apps/
│   ├── docker-compose.yml
│   └── .env

```bash
docker network create --driver bridge --subnet 172.25.0.0/16 traefik_proxy
docker network create vpn_network
```

### docker compose

Tout est défini dans un seul fichier `docker-compose.yml` et un fichier `.env`:

```yaml
networks:
  vpn:
    external: true
  traefik:
    external: true

services:
  # ============================================================
  # publicaddr : DynDNS automatique pour OVH
  # ============================================================
  publicaddr_ovhcloud:
    image: dmachard/publicaddr-ovhcloud:${PUBLICADDR_OVHCLOUD_VERSION}
    container_name: publicaddr
    env_file:
      - .env
    environment:
      PUBLICADDR_OVHCLOUD_UPDATE: ${PUBLICADDR_REFRESH}
      PUBLICADDR_OVHCLOUD_DEBUG: ${PUBLICADDR_DEBUG}
      PUBLICADDR_OVHCLOUD_HAS_IPV6: ${PUBLICADDR_HAS_IPV6}
      PUBLICADDR_OVHCLOUD_ZONE: ${PUBLICADDR_ZONE}
      PUBLICADDR_OVHCLOUD_SUBDOMAINS: ${PUBLICADDR_SUBDOMAINS}
      PUBLICADDR_OVHCLOUD_ENDPOINT: ${OVH_EP}
      PUBLICADDR_OVHCLOUD_APPLICATION_KEY: ${OVH_AK}
      PUBLICADDR_OVHCLOUD_APPLICATION_SECRET: ${OVH_AS}
      PUBLICADDR_OVHCLOUD_CONSUMER_KEY: ${OVH_CK}
    restart: unless-stopped

  # ============================================================
  # dnscollector : Collecte des logs DNS (optionnel)
  # ============================================================
  dnscollector:
    image: dmachard/dnscollector:${DNSCOLLECTOR_VERSION}
    restart: unless-stopped
    container_name: dnscollector
    network_mode: host
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - ./dnscollector/config.yml:/etc/dnscollector/config.yml
      - ./dnstap:/var/dnscollector/

  # ============================================================
  # blocklist_updater : Télécharge les listes de domaines à bloquer
  # ============================================================
  blocklist_updater:
    image: alpine:latest
    container_name: blocklist_updater
    volumes:
      - ./dnsdist/:/config/
    restart: always
    command: >
        sh -c "
        apk add --no-cache curl &&
        echo '0 1 * * * curl -fsSL -o /config/blocklist.cdb https://raw.githubusercontent.com/dmachard/blocklist-domains/data/blocklist.cdb' > /etc/crontabs/root &&
        curl -fsSL -o /config/blocklist.cdb https://raw.githubusercontent.com/dmachard/blocklist-domains/data/blocklist.cdb &&
        crond -f
        "

  # ============================================================
  # dnsdist : Proxy DNS avec filtrage
  # ============================================================
  dnsdist:
    image: powerdns/${DNSDIST_VERSION}
    restart: unless-stopped
    container_name: dnsdist
    volumes:
      - ./dnsdist/:/etc/dnsdist/conf.d/
    network_mode: host
    depends_on:
      - dnscollector
      - blocklist_updater

  # ============================================================
  # wireguard : Serveur VPN
  # ============================================================
  wireguard:
    image: lscr.io/linuxserver/wireguard:${WIREGUARD_VERSION}
    container_name: wireguard
    cap_add:
      - NET_ADMIN
    env_file:
      - .env
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
      - SERVERURL=${WIREGUARD_SERVERURL}
      - SERVERPORT=${WIREGUARD_SERVERPORT}
      - PEERS=${WIREGUARD_PEERS}
      - PEERDNS=${WIREGUARD_PEERDNS}
      - INTERNAL_SUBNET=${WIREGUARD_INTERNAL_SUBNET}
      - ALLOWEDIPS=${WIREGUARD_ALLOWEDIPS}
      - PERSISTENTKEEPALIVE_PEERS=all
      - LOG_CONFS=true
    volumes:
      - ./wireguard:/config
      - /lib/modules:/lib/modules
    ports:
      - 51820:51820/udp
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
    networks:
      - vpn
    restart: unless-stopped

  # ============================================================
  # Traefik : Reverse proxy
  # ============================================================
  traefik:
    image: traefik:${TRAEFIK_VERSION}
    container_name: traefik
    ports:
      - 443:443/tcp
    networks:
      - traefik
    restart: unless-stopped
    labels:
      - traefik.enable=true
      - traefik.http.routers.traefik-dashboard.entrypoints=websecure
      - traefik.http.routers.traefik-dashboard.rule=Host(`traefik.myhomelab.tld`)
      - traefik.http.routers.traefik-dashboard.service=api@internal
      - traefik.http.routers.traefik-dashboard.tls.certresolver=letsencrypt
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./certs:/data
    env_file:
      - .env
    environment:
      - "OVH_ENDPOINT="${OVH_EP}"
      - "OVH_APPLICATION_KEY="${OVH_AK}"
      - "OVH_APPLICATION_SECRET="${OVH_AS}"
      - "OVH_CONSUMER_KEY="${OVH_CK}"
    command:
      # See https://doc.traefik.io/traefik/providers/docker/
      - --providers.docker=true
      - --providers.docker.network=traefik
      - --providers.docker.watch=true
      - --providers.docker.exposedbydefault=false
      # See https://doc.traefik.io/traefik/observability/logs/
      - --log.filePath=/data/traefik.log
      - --log.level=DEBUG
      - --accesslog
      # See https://doc.traefik.io/traefik/operations/api/
      - --api=true
      - --api.dashboard=true
```

**Env file**

```
# Software version to use
DNSCOLLECTOR_VERSION="dnsdist-20:v2.0.0"
DNSDIST_VERSION="v2.0.0"
TRAEFIK_VERSION="v3.6.7"
WIREGUARD_VERSION="1.0.20250521-r1-ls101"
PUBLICADDR_OVHCLOUD_VERSION="v0.10.0"

# Fréquence de mise à jour (en secondes)
PUBLICADDR_UPDATE: 300
PUBLICADDR_DEBUG: 0
PUBLICADDR_HAS_IPV6: 0

# domaine dns à mettre à jour
PUBLICADDR_ZONE: myhomelab.tld
PUBLICADDR_SUBDOMAINS: *.myhomelab.tld

# credentials ovh https://eu.api.ovh.com/createToken/
OVH_EP: ovh-eu
OVH_AK: votre_application_key
OVH_AS: votre_application_secret
OVH_CK: votre_consumer_key

# domaine et port du vpn
WIREGUARD_SERVERURL=vpn.myhomelab.tld
WIREGUARD_SERVERPORT=51820

# Liste des clients qui auront accès au VPN
WIREGUARD_PEERS=denis_mobile

# dns à utiliser par les clients
WIREGUARD_PEERDNS=172.16.0.253

# Réseau privé du VPN (10.13.13.0/24)
WIREGUARD_INTERNAL_SUBNET=10.13.13.0

# Plages IP routées via le VPN (split-tunneling)
WIREGUARD_ALLOWEDIPS=172.16.0.0/12
```

**Le script download_blocklist.sh** télécharge des listes de domaines publicitaires.

```bash
#!/bin/sh
curl -o /config/blocklist.cdb https://raw.githubusercontent.com/dmachard/blocklist-domains/data/blocklist.cdb
```

**Configuration de dnsdist** (`dnsdist/dnsdist.yml`) :

```
****
```

## Conclusion

Cette configuration tourne de manière stable depuis plusieurs anénes sans intervention. Les certificats se renouvellent automatiquement, le DNS se met à jour tout seul, et les containers redémarrent en cas de reboot.
Mon téléphone garde le VPN WireGuard activé en permanence. J'accède à mes photos via Immich comme si c'était un service cloud, mais tout reste chez moi avec un contrôle total.