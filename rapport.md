# Rapport

## Auteurs

**Verhaege Lucien**

**Milan Delzenne**

## Un peu de vocabulaire

**phys**: représente la machine physique.

**virtu**: représente la machine de virtualisation.

**vm**: représente la machine virtuelle.

**vm-ip**: l'adresse IP de la machine virtuelle.

**login**: notre identifiant sur le réseau (*prenom.nom.etu*).

**root**: l'utilisateur root (logique), en opposition à...

**...user**: l'utilisateur non root de la machine virtuelle.

# Partie 1

## 1. Se connecter à la machine de virtualisation avec ssh
La machine physique sur laquelle on travaille n'est pas la même que la machine de virtualisation qui nous a été attribuée. Pour s'y connecter, on effectue la commande ssh comme suit:

    login@phys$ ssh virtu.iutinfo.fr

**Attention**, si c'est la première fois qu'on essaie de se connecter à cette machine en ssh, on aura un message similaire à ce qui suit:

    The authenticity of host 'teck03.iutinfo.fr (172.18.48.223)' can't be established.
    ED25519 key fingerprint is SHA256:sefLeWVX2+oaIsGPjmgMhGfaQNF6UmGOAxJD4TBGnEI.
    This key is not known by any other names
    Are you sure you want to continue connecting (yes/no/[fingerprint])?

Dans ce cas, il est **primordial** de vérifier que l'empreinte indiquée par le terminal correspond bien à celle fournie par l'administrateur système.

Le terminal demande ensuite un mot de passe qu'il faudra donner à chaque fois qu'on essaiera de se connecter en ssh à cette machine.

## 2. Faciliter la connexion
Il est possible de faciliter la connexion ssh en créant et utilisant une paire de clés: une clé *publique*, partagée sur le réseau ssh, et une clé *privée*, connue uniquement par celui qui l'a créée.

Deux étapes sont nécessaires pour leur utilisation.

### 2.1 Créer les clés
Il faut d'abord créer les clés, ce qui se fait grâce à la commande:

    login@phys$ ssh-keygen

Il est possible de choisir où cette clé sera enregistrée sur notre machine, par défaut elle est enregistrée sous `/home/infoetu/login/.ssh/id_rsa`.

### 2.2 Partager la clé publique
Il faut ensuite partager la clé publique sur le serveur. Pour ce faire, on utilise la commande:

    login@phys$ ssh-copy-id -i .ssh/id_rsa.pub login@virtu


## 3. Créer et gérer des machines virtuelles
Nous allons voir comment gérer les machines virtuelles. Pour cela, nous allons utiliser un script nommé `vmiut` sur la machine de virtualisation. Pour pouvoir l'utiliser, il faut **impérativement** commencer par utiliser la commande suivante:

    login@virtu$ source /home/public/vm/vm.env

### 3.1 Créer et supprimer une machine virtuelle
Pour créer la machine virtuelle `vm`:

    login@virtu$ vmiut creer vm

Pour la supprimer:

    login@virtu$ vmiut supprimer vm

### 3.2 Lister les machines virtuelles
Pour lister les machines virtuelles présentes sur la machine de virtualisation:

    login@virtu$ vmiut lister


### 3.3 Démarrer et arrêter une machine virtuelle
Pour démarrer la machine virtuelle `vm`:

    login@virtu$ vmiut demarrer vm

Pour l'arrêter (**attention**, à faire à chaque fois qu'on a fini de l'utiliser):

    login@virtu$ vmiut arreter vm


### 3.4 Obtenir des informations sur la machine virtuelle
Pour ce faire:

    login@virtu$ vmiut info vm

### 3.5 Se connecter à la machine virtuelle
2 possibilités:

#### 3.5.1 Console virtuelle
Depuis la machine physique:

    login@phys$ ssh -X frene20

Puis, une fois sur la machine de virtualisation:

    login@virtu$ vmiut console vm

#### 3.5.2 Connexion en ssh
Depuis la machine de virtualisation:

    login@virtu$ ssh user@vm-ip

On peut obtenir `vm-ip` avec `vmiut info vm`, voir "**3.4 Obtenir des informations sur la machine virtuelle**".

---
Il est préférable que la machine virtuelle ait une adresse IP statique. Pour ce faire, on se commence par se connecter à la machine virtuelle en tant que root (nom d'utilisateur: *root* / mot de passe: *root*).

Avant tout changement, on doit couper l'interface réseau:

    root@vm# ifdown enp0s3

Ensuite, on ouvre le fichier `interfaces`. C'est ce fichier qu'on va modifier pour assigner une adresse IP statique à notre machine virtuelle et au routeur.

    root@vm# nano /etc/network/interfaces

Dans ce fichier, on supprime le `dhcp` présent sur la dernière ligne et on le remplace par:

     iface enp0s3 inet static
		address 192.168.194.3/24
		gateway 192.168.194.2

Ensuite, pour assigner une adresse statique au serveur DNS, on ouvre le fichier `resolv.conf` comme suit:

    root@vm# nano /etc/resolv.conf

Et on écrit sur la dernière ligne:

    nameserver 192.168.194.2

On peut vérifier les adresses avec les commandes:

1. `root@vm# ifup enp0s3` pour rallumer l'interface

2. `root@vm# ip addr show`

3. `root@vm# ip route show`

4. `root@vm# reboot` pour vérifier que les modifications persistent après un redémarrage de la machine virtuelle.

## 4. Configurer et mettre à jour la machine virtuelle

### 4.1 Connexion root en ssh

Pour se connecter en root sur la machine virtuelle via ssh, deux étapes. D'abord, se connecter en ssh à la machine virtuelle sur user:

    login@virtu ssh user@vm-ip

Puis, on utilise la commande `su` pour passer sur root:

    user@vm# su -l root

### 4.2 Accès extérieur pour la machine virtuelle

Pour pouvoir accéder au web depuis la machine virtuelle, on commence par se connecter en **root**.

Ensuite, on effectue la commande suivante:

    root@vm# nano /etc/environnement

Dans le fichier ainsi ouvert, on ajoute:

    HTTP_PROXY=http://cache.univ-lille.fr:3128
    HTTPS_PROXY=http://cache.univ-lille.fr:3128
    http_proxy=http://cache.univ-lille.fr:3128
    https_proxy=http://cache.univ-lille.fr:3128
    NO_PROXY=localhost,192.168.194.0/24,172.18.48.0/22

### 4.3 Mise à jour de la machine virtuelle

Pour mettre à jour la machine virtuelle, effectuer depuis root:

    root@vm# apt update && apt full-upgrade

On va à un moment nous demander de choisir entre `/dev/sda` et `/dev/sda1`. On choisit `/dev/sda` en appuyant sur la barre espace et on appuie sur 'entrée' pour valider.

    [*]/dev/sda
    [ ]/dev/sda1

## 5. Installation d'outils

Pour installer des logiciels:

    root@vm# apt nom-logiciel

...où `nom-logiciel` est le nom du logiciel à installer.

## 6. Création d'alias

Pour rendre plus simple et plus rapide la connexion ssh à la machine virtuelle, on peut créer des alias sur la machine physique.

Pour ce faire: commencer par ouvrir `config` depuis la machine physique.

    login@phys$ nano .ssh/config

Dans le fichier ainsi ouvert, ajouter:

    Host virt
            HostName virtu.iutinfo.fr
            User login
    
    Host vm
            User user
            HostName 192.168.194.3
    
    Host vmjump
            User user
            HostName 192.168.194.3
            Proxyjump virt

**Attention**: on n'oublie pas d'effectuer la commande

    login@virtu$ ssh-copy-id -i .ssh/id_rsa.pub user@192.168.194.3

depuis la machine de virtualisation (voir "**2. Faciliter la connexion**" pour plus d'informations).

Grâce à cette manipulation, il suffira d'écrire, sur la machine physique, la commande: 
    
    login@phys$ ssh virt

pour se connecter en ssh à la machine de virtualisation, et:

    login@phys$ ssh vm

pour se connecter en ssh à la machine virtuelle sur user.

# Partie 2

## Changer le nom d'une machine

Pour changer le nom d'une machine il faut modifier le fichier /etc/hostname en mettant le nom voulu dedans.

Pour changer le nom du localhost en le nom de votre machine, il faut associer le nom voulu et l'adresse IP de la manière suivante dans le fichier /etc/hosts (par exemple si on veut changer le localhost en matrix):

    127.0.0.1 matrix

## Installer et utiliser sudo

La commande sudo permet d'exécuter des commandes en tant que root.

Pour installer sudo :

    root@vm# apt install sudo

Pour permettre à un user d'utiliser la commande root :

    root@vm# usermod -aG sudo user

## Configurer la synchronisation de l'horloge

Afin de vérifier les évènements système du protocole NTP qui permet de synchroniser l'horloge, on peut utiliser la commande :

    root@vm# journalctl -u systemd-timesyncd

Pour configurer l'horloge de la machine avec un serveur ntp, il faut accéder au fichier /etc/systemd/timesyncd.conf et le modifier ainsi :

    NTP=ntp.univ-lille.fr

Il faudra ensuite redémarrer le service timesyncd en faisant la commande :

    root@vm# systemctl restart systemd-timesyncd.service

Si la configuration ne fonctionne toujours pas lorsqu'on tape la commande :

    root@vm# timedatectl status

On active la synchronisation ntp à l'aide de la commande :

    root@vm# timedatectl set-ntp true

**Ne pas oublier de redémarrer le service timesyncd une fois l'opération effectuée**

## Mise en place d'un serveur postegresql

Pour l'installer :

    root@vm# apt install postgresql

On s'assure que le service de postgresql est bien démarré avec la commande :

    root@vm# sudo /etc/init.d/postgresql restart

Pour faire des actions sur la base de données en tant qu'administrateur, il faut se connecter sur l'utilisateur postgres :

    root@vm# su -l postgres

On crée ensuite un nouvel utilisateur (ici il sera superuser et on initialisera directement le mot de passe) :

    postgres@vm$ createuser -P -s -e user

On crée ensuite une nouvelle base de donnée (ici le propriétaire de la base est spécifié dans la commande avec l'option "-O") :

    postgres@vm$ createdb -O user dbname

Pour se connecter à la base de données une fois qu'elle a été créée, il faut taper la commande suivante (ici on suppose que l'utilisateur avec lequel on veut se connecter à la base de données s'appelle user et que le nom de la base de données est dbname):

    user@vm$ psql -h localhost -U user dbname

Vous pouvez maintenant utiliser posqtgresql normalement et gérer votre base de données.

### Voici un rappel des quelques commandes importantes en sql.

- Créer une nouvelle table (ici on créé une table name contenant une colonne column de type text):
```
    create table name(column text);
```
- Insérer des données dans une table (ici on insère une ligne contenant un mot word dans la table name créée plus haut):
```
    insert into name values('word');
```
- Faire une requête simple (ici on affiche toutes les entrées de la table name créée plus haut):
```
    select * from name;
```

# Partie 3

## 1. Accès à un service HTTP sur la VM

### 1.1 Un premier service pour tester

Pour installer nginx:

    root@vm# apt install nginx

Vérifier que nginx est démarré:

    root@vm# systemctl | grep nginx

Pour installer curl:

    root@vm# apt install curl

On peut vérifier que l'on peut bien accéder à `nginx` grâce à la commande:

    user@vm# curl http://localhost

On peut également essayer d'y accéder depuis la machine de virtualisation, en remplaçant 'localhost' par l'adresse IP de la machine virtuelle:

    login@virtu$ curl http://192.168.194.26

Mais ça ne fonctionnera pas. Il faut utiliser l'option -L de curl qui permet de faire un tunnel SSH vers le réseau ciblé. Ici:

    login@virtu$ curl -L 0.0.0.0:80 192.168.194.3:80

`0.0.0.0` est la route par défaut, c'est-à-dire la route qui mène "au reste d'internet" plutôt qu'autre part sur le réseau local.

### 1.2 Accès au service depuis la machine physique

On effectue la commande:

    login@virtu$ ssh -L 0.0.0.0:80:vm-ip:80 user@vm-ip

## 2. Installation de Synapse

### 2.1 Installation du paquet sous Debian

Pour installer Synapse sur la machine virtuelle, on effectue les 4 commandes suivantes (extraites de la [documentation Synapse](https://matrix-org.github.io/synapse/latest/setup/installation.html#matrixorg-packages)):

    root@vm# apt install -y lsb-release wget apt-transport-https
    root@vm# wget -O /usr/share/keyrings/matrix-org-archive-keyring.gpg https://packages.matrix.org/debian/matrix-org-archive-keyring.gpg
    root@vm# echo "deb [signed-by=/usr/share/keyrings/matrix-org-archive-keyring.gpg] https://packages.matrix.org/debian/ $(lsb_release -cs) main" |
        tee /etc/apt/sources.list.d/matrix-org.list
    root@vm# apt update
    root@vm# apt install matrix-synapse-py3

Au cours de l'installation de Synapse, il va nous être demandé le nom de notre instance. Il faut alors indiquer:

    virtu.iutinfo.fr:8008

Et remplacer virtu par la machine de virtualisation (`frene20` par exemple).

### 2.2 Paramétrage spécifique pour une instance dans un réseau privé

On ne veut pas que notre serveur contacte d’autres serveurs pour obtenir des clés publiques de signatures. Pour ce faire, on va appliquer la documentation en affectant la variable de configuration `trusted_key_servers` à `[]`.

On ouvre le fichier de configuration à modifier avec:

    root@matrix# nano /etc/matrix-synapse/homeserver.yaml

Enfin, dans ce fichier, on remplace la ligne:

    trusted_key_servers:

par:

    trusted_key_servers: []

### 2.3 Utilisation d'une base Postgres

Par défaut, Synapse utilise une base de données au format `sqlite`, ce qui n'est pas suffisant pour nos besoins. On préfère utiliser le format `postgres`.

Pour cela, on retourne dans le fichier de configuration de Synapse (`/etc/matrix-synapse/homeserver.yaml`) et dans la partie `database`, on met:

    database:
        name: psycopg2
        args:
            user: <user>
            password: <pass>
            database: <db>
            host: <host>
            cp_min: 5
            cp_max: 10


On veut ensuite créer un utilisateur et une base de données. On se connecte d'abord sur l'utilisateur postgres:

    root@vm# su - postgres

Notre base de données précédemment créée ne convenant pas à Synapse, on la supprime avec:

    postgres@vm# dropdb database_name

On doit ensuite créer un utilisateur Synapse et une base de données grâce à:

    createuser --pwprompt synapse_user
    createdb --encoding=UTF8 --locale=C --template=template0 --owner=synapse_user database_name

### 2.4 Création d’utilisateurs

On veut créer 2 utilisateurs partageant la même clé d'enregistrement. On va pour cela utiliser la commande `register_new_matrix_user` avec l'option `-k`.

