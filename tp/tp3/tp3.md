# Créer et utiliser un premier playbook avec plusieurs tâches

Dans ce 3eme TP nous allons créer un playbook pour l'hote 03 qui installera un serveur web LAMP (Linux Apache MySQL PHP)

L'objectif est d'automatiser l'installation de ce serveur, pour pouvoir aussi l'installer sur host 01 (pour host 02 vu que c'est CentOS nous verrons cela dans le TP suivant pour rendre variable certaines partie de notre playbook)

Voici donc les différentes tâches que nous allons effectuer :

- Côté serveur web :
- - Installer les packages apache2, php et php-mysql
- - Déployer les sources de notre application dans notre serveur web distant
- - S'assurer que le service apache est bien démarré
- Côté serveur base de données :
- - Installer les packages mysql
- - Modifier le mot de passe root
- - Autoriser notre serveur web à communiquer avec la base de données
- - Configurer notre table mysql avec les bonnes colonnes et autorisations


## Préparation

/!\ Toutes les commandes ne sont pas présentes, à vous de les retrouver avec ce qui a déjà été fait

- Pour ce nouveau TP nous allons recréer un environnement propre, à partir du docker compose qui est dans le dossier tp3: 
```
docker-compose up -d
```
- Il est nécessaire de se connecter à master01 et de générer une clé ssh, puis de la déployer sur les deux machines (webserver et dbserver )
- Puis créer un fichier hosts sur la machine master01 pour déclarer nos deux hôtes (webserver et dbserver) dans le dossier tp3
- Il faut également modifier la configuration d'ansible pour que le fichier hosts utilisé soit celui du dossier courant. Créer dans le dossier tp3 un fichier ansible.cfg avec le contenu suivant :
```
[defaults]
inventory       = ./hosts
```
- Il est temps maintenant de créer notre fichier playbook : dans le dossier tp3 créer un fichier playbook.yml 

## Arborescence :
- Différents fichiers et dossiers sont à votre disposition :
```
|── files
│   └── app
│       ├── index.php
│       └── validation.php
|       └── db-config.php
|       └── table.sql
|
|── docker-compose.yml
|── hosts
|── ansible.cfg
|── playbook.yml
```
- Les fichiers dans files/app correspondent à l'application qui doit être déployée
- Le fichier docker-compose correspond à l'environnement d'éxecution 
- Le fichier hosts contient l'inventaire de ce projet
- Le fichier ansible.cfg contient la surcharge de configuration qui a été faite 

## Serveur Web
- Il est temps d'alimenter notre playbook :) 
- Cibler le bon hote l'hote webserver :
```
host: webserver
```
- Ensuite on déclare une tache pour installer les paquets nécessaires :
```
name: install apache and php last version
  apt:
    name:
      - apache2
      - php
      - php-mysql
    state: present
	update_cache: yes
```
- Puis on assigne les droits en écritures sur le dossier web :
```
name: Give writable mode to http folder
    file:
      path: /var/www/html
      state: directory
      mode: '0755'
```
- On supprime le fichier index par défaut :
```
name: remove default index.html
    file:
      path: /var/www/html/index.html
      state: absent
```
- On envoie les fichiers web :
```
name: upload web app source
    copy:
      src: files/app/
      dest: /var/www/html/
```
- On envoie le fichier de config db-config.php (avant de l'envoyer, le modifier pour remplacer les variables par les bonnes valeurs)
```
  - name: deploy php database config
    copy:
      src: "files/db-config.php"
      dest: "/var/www/html/db-config.php"
```
- On démarre le service apache
```
name: ensure apache service is start
    service:
      name: apache2
      state: started
      enabled: yes
```
-> Le serveur web est prêt à être déployé passons à la base de données

## Serveur de BDD
- Configurer l'environnement de l'hote dans le fichier playbook.yml :
```
hosts: dbserver
vars_files: vars/main.yml
```
- Créer le fichier vars/main.yml avec les variables suivantes (à adapter à votre config) :
```
mysql_user: "admin"
mysql_password: "secret"
mysql_dbname: "blog"
db_host: "192.168.0.22"
webserver_host: "192.168.0.21"
root_password: "mypassword"
```
- Installer mysql :
```
tasks:
  - name: install mysql
    apt:
      name: 
        - mysql-server
        - python3-pip
        - python3-setuptools
        - python3-pymysql      
      state: present
      update_cache: yes  
```
- On installe un paquet python nécessaire au bon fonctionnement des modules mysql pour ansible :
```
  - name: Install the Python MySQLB module
    pip: name=mysql-connector
```
- Créer la configuration du client mysql:
```
name: Create MySQL client config
    copy:
      dest: "/root/.my.cnf"
      content: |
        [client]
        user=root
        password={{ root_password }}
      mode: 0400
```
- Autoriser les connexions à mysql :
```
  - name: Allow external MySQL connexions 
    lineinfile:
      path: /etc/mysql/mysql.conf.d/mysqld.cnf
      regexp: '^bind-address'
      line: "# bind-address"
    notify: Restart mysql
```
- Charger la table sql (remplacer les variables par les bonnes valeurs en dur dans le fichier table.sql)
```
  - name: upload sql table config
    copy:
      src: "files/table.sql"
      dest: "/tmp/table.sql"
```
- On initialise le home de mysql et on démarre le service (via shell car service ne fonctionne pas sur docker pour démarrer mysql)
```
  - name : moving mysql home
    user:
      name: mysql
      home: /var/lib/mysql

  - name: start mysql
    shell:
      service mysql start
```
- On importe la base de données :
```
  - name: add sql table to database
    mysql_db:
      name: "{{ mysql_dbname }}"
      state: present
      login_user: root
      login_password: '{{ root_password }}'
      state: import 
      target: /tmp/table.sql
```
- Créer un utilisateur mysql :
```
  - name: "Create {{ mysql_user }} with all {{ mysql_dbname }} privileges"
    mysql_user:
      name: "{{ mysql_user }}"
      password: "{{ mysql_password }}"
      priv: "*.*:ALL"
      state: present
      login_user: root
      login_password: '{{ root_password }}'
      login_unix_socket: /var/run/mysqld/mysqld.sock
```
-> Le deploiement de notre serveur de base de données est prêt

## Lancer notre playbook
- vérifier la syntaxe du playbook :
```
ansible-playbook playbook.yml --syntax-check
```
- Lancer notre playbook :
```
ansible-playbook playbook.yml
```
- Vérifier sur localhost:8042 si notre application web s'affiche et si l'on peut enregistrer un article (ce qui signifie que la connexion avec la BDD fonctionne)
