# Exploration de la galaxie Ansible, téléchargement et utilisation de modules, ajout de tags dans un playbook.

Ce TP va permettre de poursuivre le travail commencé sur l'applications web.
Nous allons donc  
- Remplacer les taches de notre play DBserver par un rôle existant
- tagguer nos taches pour savoir ce qui est de l'installation, de la configuration, de la gestion de service ou du déploiement
- Creer un rôle pour notre webserver et le publier sur ansible galaxy

## Utilisation d'un rôle galaxy
- Reprenez le playbook du tp précédent (ou celui fournit dans le dossier corriger)
- Nous allons travailler avec ce rôles :
https://galaxy.ansible.com/geerlingguy/mysql
* Installer le rôle
```
ansible-galaxy install geerlingguy.mysql
```
- Suivre les instructions données la définition des variables
- Ajouter au playbook le rôle installé et supprimer (ou commenter) toutes les taches du play DBSERVER, sauf celles concernant l'import du fichier table.sql
```
- hosts: dbserver
  vars_files:
    - vars/main.yml
  name: installation du role mysql
  roles:
    - { role: geerlingguy.mysql }
  #become: yes
```
* Penser à définir les paramètres suivants dans le fichier vars/main.yml pour le rôle

```
mysql_python_package_debian:
  - python3-dev
  - libmysqlclient-dev
  - python3-pip 
  - python3-setuptools
  - python3-pymysql

mysql_dbname: db_demo

mysql_root_home: /root
mysql_root_username: root
mysql_root_password: rootpassword
mysql_root_password_update: yes
mysql_databases:
  - name: example_db
    encoding: latin1
    collation: latin1_general_ci
mysql_users:
  - name: example_user
    host: "%"
    password: similarly-secure-password
    priv: "example_db.*:ALL"
```


- Tester en relançant le playbook pour vérifier si cela fonctionne toujours

## Ajouter des tags à nos taches
* Ajouter sur nos différentes taches et play les tags suivants :
  * install
  * config
  * service
  * deploiement
  * db
  * web
* Lancer le playbook uniquement pour ce qui concerne l'installation
* Puis lancer le en omettant les taches d'installation

## Création et publication d'un rôle
- Initialiser un role appeler webserver-blog
```
ansible-galaxy init webserver-blog 
```
- Déplacer les différents éléments dans les fichiers correspondant du template généré
- Pousser le code sur un nouveau dépôt Github 
- Connectez vous sur galaxy.ansible.com (avec un compte github à créer si vous n'en avez pas)
- Publier votre role : dans le menu rôle cliquer sur rôle et vous avez un bouton Import : remplissez le formulaire pour importer avec l'adresse du dépôt Git. 
- Importer dans le playbook original le rôle crée et supprimer les tâches liées au serveur web
- Récupérer le rôle créé par un autre stagiaire
- Inclure ce rôle à la place du votre et lancer le playbook

-> Vous savez maintenant utiliser les rôles et les tags dans vos playbook et partager votre travail
