---
title: "Guide installation de guacamole en mode container"
summary: "Guide d'installation de guacamole"
date: 2019-01-29T00:00:00+01:00
draft: false
tags: ['docker', 'guacamole', 'installation']
---

# Guide installation de guacamole en mode container

Guide d'installation de **[guacamole](https://guacamole.apache.org/)** en mode container avec une base de donnée PostgreSQL.

## Prérequis

Ce tutorial nécessite un environnement docker et de récupérer les images dockers suivantes:

```bash
sudo docker pull postgres:9.4
sudo docker pull guacamole/guacd
sudo docker pull guacamole/guacamole
```

## Déploiment PostgreSQL

### Démarrage container

Pour avoir des données persistantes, prévoir un répertoire sur le système hôte.

```bash
mkdir -p /home/guacamole/postgresql_data
cd /home/guacamole/postgresql_data
sudo docker container run --name postgres01 -e POSTGRES_PASSWORD=bonjour -d -p 5432:5432 -v $PWD/postgresql_data:/var/lib/postgresql/data  postgres

### Vérification des logs

```bash
sudo docker logs postgres01
The files belonging to this database system will be owned by user "postgres".
This user must also own the server process
...
LOG:  database system is ready to accept connections
```

### alter user 

```bash
sudo docker exec -it postgres01 bash
root@c26e9831087a:/# su - postgres
postgres@c26e9831087a:~$  psql -d postgres -c "ALTER USER postgres WITH PASSWORD 'bonjour';"
ALTER ROLE
```

### test de connexion

Enfin faire un test de connexion sur la bdd 

```bash
sudo apt-get install -y postgresql-client
psql -h localhost -U postgres -d postgres
Password for user postgres: 
psql (13.4 (Ubuntu 13.4-0ubuntu0.21.04.1), server 14.0 (Debian 14.0-1.pgdg110+1))
WARNING: psql major version 13, server major version 14.
         Some psql features might not work.
Type "help" for help.

postgres=#
```

## Déploiment Guacamole

### Création base

```bash
psql -h localhost -U postgres -d postgres

CREATE USER guacamole_user SUPERUSER PASSWORD 'guac';
CREATE DATABASE guacamole_db ;
GRANT ALL PRIVILEGES ON DATABASE guacamole_db TO guacamole_user;
```

### Test connexion

```bash
$ psql -h localhost -U guacamole_user -d guacamole_db
Password for user guacamole_user: 
psql (13.4 (Ubuntu 13.4-0ubuntu0.21.04.1), server 14.0 (Debian 14.0-1.pgdg110+1))
WARNING: psql major version 13, server major version 14.
         Some psql features might not work.
Type "help" for help.

guacamole_db=# exit
```

### Création schéma

```bash
sudo docker run --rm guacamole/guacamole /opt/guacamole/bin/initdb.sh --postgres > initdb.sql
cat initdb.sql | psql -h localhost -U guacamole_user -d guacamole_db -f -
Password for user guacamole_user: 
CREATE TYPE
....
INSERT 0 6
INSERT 0 3
```

### Démarrage des containers

Lancement du serveur guacd

```bash
sudo docker run --name guacd01 -d guacamole/guacd
```

Vérification

```bash
$ sudo docker logs guacd01
guacd[7]: INFO:	Guacamole proxy daemon (guacd) version 1.3.0 started
guacd[7]: INFO:	Listening on host 0.0.0.0, port 4822
```

Lancement du client guacamole

```bash
sudo docker container run --name guacamole01 --link guacd01:guacd --link postgres01:postgres -e POSTGRES_DATABASE=guacamole_db -e POSTGRES_USER=guacamole_user -e POSTGRES_PASSWORD=guac -d -p 8080:8080 guacamole/guacamole
```

Vérification

```bash
$ sudo docker logs guacamole01
05-Nov-2021 19:38:11.788 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Server version name:   Apache Tomcat/8.5.72
05-Nov-2021 19:38:11.796 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Server built:          Oct 1 2021 15:15:33 UTC
05-Nov-2021 19:38:11.797 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Server version number: 8.5.72.0
...
at/webapps/guacamole.war] has finished in [6,039] ms
05-Nov-2021 19:38:18.231 INFO [main] org.apache.coyote.AbstractProtocol.start Starting ProtocolHandler ["http-nio-8080"]
05-Nov-2021 19:38:18.245 INFO [main] org.apache.catalina.startup.Catalina.start Server startup in 6168 ms
```

Vérification de l'écoute du programme

```bash
$ sudo netstat -anp | grep 8080
tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN      2275956/docker-prox 
tcp6       0      0 :::8080                 :::*                    LISTEN      2275964/docker-prox
```

## Test de connexion

L'interface de guacamole est accessible avec l'adresse suivante: http://ip:8080/guacamole/

Le compte par défaut:
- login=guacadmin
- password=guacadmin