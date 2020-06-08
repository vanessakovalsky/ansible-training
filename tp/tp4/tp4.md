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
ansible-galaxy install vanessakovalsky.mysql
```
- Suivre les instructions données la définition des variables
- Ajouter au playbook le rôle installé et supprimer (ou commenter) toutes les taches du play DBSERVER, sauf celles concernant l'import du fichier table.sql
```
- hosts: dbserver
  vars_files:
    - vars/main.yml
  roles:
    - { role: vanessakovalsky.mysql }
```
- Tester en relançant le playbook pour vérifier si cela fonctionne toujours

## Ajouter des tags à nos taches
- Ajouter sur nos différentes taches et play les tags suivants :
-- install
-- config
-- service
-- deploiement
-- db
-- web
- Lancer le playbook uniquement pour ce qui concerne l'installation
- Puis lancer le en omettant les taches d'installation

## Création et publication d'un rôle
- Initialiser un role appeler webserver-blog
```
ansible-galaxy init webserver-blog 
```
- Déplacer les différents éléments dans les fichiers correspondant du template généré
- Importer dans le playbook original le rôle crée et supprimer les tâches liées au serveur web
- Connectez vous sur galaxy.ansible.com (avec un compte github à créer si vous n'en avez pas)
- Publier votre role
- Récupérer le rôle créé par un autre stagiaire
- Inclure ce rôle à la place du votre et lancer le playbook

-> Vous savez maintenant utiliser les rôles et les tags dans vos playbook et partager votre travail
