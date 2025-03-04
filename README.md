# GLPI-Ansible-
how to create a glpi with file ansible 
Présentation du script Ansible pour l'installation de GLPI
Introduction
Ce script Ansible automatise l'installation et la configuration de GLPI, un logiciel open-source de gestion des services informatiques (ITSM). Il assure l'installation des dépendances, la configuration de la base de données MySQL/MariaDB, le téléchargement de GLPI et la configuration d'Apache.

Structure du script
Le fichier glpi.yml est structuré en plusieurs sections :

Définition du playbook
Installation des paquets nécessaires
Configuration de MySQL
Téléchargement et configuration de GLPI
Configuration d'Apache
Redémarrage des services
Détails du code
1️⃣ Définition du Playbook
yaml
Copier
Modifier
- name: Installation et déploiement de GLPI
  hosts: all
  become: yes
  vars:
    mysql_root_password: "toto"
    glpi_db_name: "glpi"
    glpi_db_user: "toto"
    glpi_db_password: "toto"
📌 Explication :

hosts: all → Exécute les tâches sur tous les hôtes définis dans l’inventaire.
become: yes → Obtient les privilèges administrateurs.
vars → Définit les variables pour MySQL (mot de passe root, nom de la base, utilisateur et mot de passe).
2️⃣ Installation des dépendances
yaml
Copier
Modifier
    - name: Installation des paquets
      apt:
        name: 
          - python3-pymysql
          - apache2
          - php
          - php-mysql
          - php-curl
          - php-gd
          - php-xml
          - php-mbstring
          - php-intl
          - php-zip
          - php-bz2
          - php-ldap
          - mariadb-server
        state: present
        update_cache: yes
📌 Explication :

Installe tous les paquets nécessaires pour Apache, PHP et MariaDB.
update_cache: yes → Met à jour les dépôts avant installation.
3️⃣ Configuration de MySQL
🔹 Définition du mot de passe root :
yaml
Copier
Modifier
    - name: Configuration du mot de passe root MySQL
      mysql_user:
        name: root
        host: localhost
        password: "{{ mysql_root_password }}"
        login_unix_socket: /var/run/mysqld/mysqld.sock
        state: present
        check_implicit_admin: yes
📌 Explication :

Définit le mot de passe du super utilisateur MySQL (root).
Utilise login_unix_socket pour éviter d’avoir besoin d’un mot de passe temporaire.
🔹 Création du fichier .my.cnf pour éviter de retaper le mot de passe :
yaml
Copier
Modifier
    - name: Création du fichier .my.cnf
      copy:
        content: |
          [client]
          user=root
          password={{ mysql_root_password }}
        dest: /root/.my.cnf
        mode: '0600'
📌 Explication :

Crée un fichier de configuration MySQL permettant une connexion sans mot de passe interactif.
🔹 Création de la base de données et d’un utilisateur dédié :
yaml
Copier
Modifier
    - name: Création de la base de données GLPI
      mysql_db:
        name: "{{ glpi_db_name }}"
        state: present

    - name: Création de l'utilisateur DB
      mysql_user:
        name: "{{ glpi_db_user }}"
        password: "{{ glpi_db_password }}"
        priv: "{{ glpi_db_name }}.*:ALL"
        state: present
📌 Explication :

Crée la base de données GLPI.
Crée un utilisateur avec les permissions nécessaires.
4️⃣ Téléchargement et installation de GLPI
yaml
Copier
Modifier
    - name: Téléchargement et extraction de GLPI
      unarchive:
        src: "https://github.com/glpi-project/glpi/releases/download/10.0.18/glpi-10.0.18.tgz"
        dest: "/var/www/html/"
        remote_src: yes
📌 Explication :

Télécharge et extrait GLPI dans le répertoire /var/www/html/.
🔹 Configuration des permissions :
yaml
Copier
Modifier
    - name: Configuration des permissions
      file:
        path: "/var/www/html/glpi"
        owner: www-data
        group: www-data
        recurse: yes
        mode: '0755'
📌 Explication :

Définit www-data (utilisateur d’Apache) comme propriétaire du répertoire GLPI.
Assure les permissions correctes.
5️⃣ Configuration d’Apache
yaml
Copier
Modifier
    - name: Configuration Apache
      copy:
        content: |
<VirtualHost *:80>
              ServerName glpi.local
              DocumentRoot /var/www/html/glpi
<Directory /var/www/html/glpi>
                  Options FollowSymlinks
                  AllowOverride All
                  Require all granted
</Directory>
</VirtualHost> 
        dest: /etc/apache2/sites-available/glpi.conf
📌 Explication :

Configure un VirtualHost Apache pour GLPI.
Définit glpi.local comme nom de domaine local.
🔹 Activation du site et redémarrage d’Apache :
yaml
Copier
Modifier
    - name: Activation du site
      command: a2ensite glpi.conf

    - name: Redémarrage Apache
      service:
        name: apache2
        state: restarted
📌 Explication :

Active le site avec a2ensite.
Redémarre Apache pour appliquer les changements.
