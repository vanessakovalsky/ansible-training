# Installation de l'environnement et premier pas avec Ansible

Ce premier TP va nous permettre d'avoir un environnement fonctionnel pour apprendre ansible. Pour cela on utilise des conteneur docker et l'outil docker compose. Une fois les conteneurs prêts, il sera nécessaire de générer et déployer des clés SSH pour permettre la connexion, puis sur la machine contenant ansible de faire quelques commandes pour vérifier le bon fonctionnement de notre environnement.

## Pré-requis 
- Installer Docker  : https://docs.docker.com/get-docker/
- Installer Docker-compose : https://docs.docker.com/compose/install/
- Clone le dépôt : https://github.com/vanessakovalsky/ansible-training.git 

## Lancement de notre environnement
- Se mettre dans le dossier du premier TP :
```
cd ansible-training/tp/
```
- Lancer le compose 
```
docker-compose up -d
```
- Lest 4 conteneurs (un avec ansible et 3 hôtes) sont lancer, vérifier le avec : 
```
docker container ls
```
- NB : si vous êtes sur Linux ou Mac Os il est possible d'installer ansible directement sur votre machine, mais cela demandera un peu plus de configuration côté conteneur pour autoriser la connexion SSH : https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html

## Créer et deployer la clé ssh
- Toutes les commandes se font depuis la machine de gestion (celle ou ansible est installé)
```
docker exec –t <containerID> bash
```
- On génère la clé :
```
ssh-keygen
```
- On copie la clé sur les 3 machines hôtes :
```
ssh-copy-id root@<IPmachinehost> 
ssh-copy-id root@<IPmachinehost2> 
ssh-copy-id root@<IPmachinehost3>
```
- On teste la connexion pour chaque machine : 
```
ssh root@<IPmachinehost> 
ssh root@<IPmachinehost2> 
ssh root@<IPmachinehost3>
```

## On initialise le fichier hote pour ansible 
- On créer le fichier /etc/ansible/hosts avec ce contenu (remplacer avec les IP de chaque machine):
```
[testservers] 
<IPmachinehost>
<IPmachinehost2>
<IPmachinehost3>
```
## Tester les commandes ansible :
- On test le ping :
```
ansible testservers -m ping 
```
- On liste les hôtes :
```
ansible testservers --list-hosts
```

-> Félicitation vous avez installé votre environnement Ansible.