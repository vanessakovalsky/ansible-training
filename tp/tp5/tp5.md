# Ecrire un playbook complet pour le déploiement d'un service web sur plusieurs environnements.

Ce dernier TP va permettre de rendre notre playbook prêt à être utiliser en production.
Pour cela :
- Utiliser uniquement des variables pour l'ensemble des données variables
- Adapter notre playbook à différents environnements : Debian/Ubuntu et Centos/RedHat 

## Variabiliser l'ensemble des fichiers :
- Remplacer le fichier table.sql dans file par un fichier table.sql.j2 pour le transformer en template
- A l'intérieur de se fichier remplacer le nom de la base (db_demo) par une variable que vous devez déclarer dans le fichier de variables
- Remplacer la tache : upload sql table config (côté dbserver) par :
```
  - name: upload sql table config
    template:
      src: "table.sql.j2"
      dest: "/tmp/table.sql"
```
- Cela permet d'obtenir le fichier sql charger à partir du template + des variables définies
- Même chose pour le fichier db-config.php à renommer en db-config.php.j2
- Remplacer la tâche (côté rôle webserver) deploy php database config par la tache suivante :
```
  - name: deploy php database config
    template:
      src: '{{ db_config }}'
      dest: "/var/www/html/db-config.php"
```
* N'oublier pas de modifier la variable db_config en conséquence
* Sur l'ensemble de nos tâches remplacer les valeurs par des variables et déclarer ses variables dans le vars/main.yml

## Rendre adaptable à l'environnement nos tâches :
- Nous allons utiliser les facts pour savoir dans quel type d'environnement système nous somme, pour connaitre les facts disponible :
```
ansible servers -m setup -a "filter=*os*"
```
- Nous allons remplacer les parties qui peuvent être différents, côté webserver commençons avec l'installation, remplacer la tache d'installation par :
```
  tasks:
  - name: install apache and php last version for Debian os family
    apt:
      name: ['apache2', 'php', 'php-mysql']
      state: present
      update_cache: yes
    when: ansible_facts['os_family'] == "Debian"

  - name: install apache and php last version for RedHat os family
    yum:
      name: ['httpd', 'php', 'php-mysqlnd']
      state: present
      update_cache: yes
    when: ansible_facts['os_family'] == "RedHat"
```
- Puis la partie configuration du serveur web :
```
- name: ensure apache service is start (Debian os family)
  service:
    name: apache2
    state: started
    enabled: yes
  when: ansible_facts['os_family'] == "Debian"

- name: ensure apache service is start (RedHat os family)
  service:
    name: httpd
    state: started
    enabled: yes
  when: ansible_facts['os_family'] == "RedHat"

- name: enable connection with remote database (RedHat os family)
  shell: setsebool -P httpd_can_network_connect_db 1
  when: ansible_facts['os_family'] == "RedHat"
```
- Enfin pour aller plus loin, utilisons finalement une boucle pour gérer nos packages à installer, et déclarons ces packages dans une variable :
```
- name: install extra packages (Debian os family)
  apt:
    name: "{{ extra_packages_debian }}"
    state: present
  when: ansible_facts['os_family'] == "Debian"

- name: install extra packages (RedHat os family)
  yum:
    name: "{{ extra_packages_redhat }}"
    state: present
  when: ansible_facts['os_family'] == "RedHat"
```
- La différence d'environnement est déjà gérer pour la partie BDD avec l'utilisation du rôle mysql
- Il ne reste plus qu'à tester tout ça
```
ansible-playbook playbook.yml
```

-> Félicitations vous avez terminé l'intégralité des exercices, quel sera votre prochain défi à relever avec Ansible?
