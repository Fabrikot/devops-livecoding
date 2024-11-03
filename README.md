# devops-livecoding

Bienvenue sur mon README 🥳

# Questions

#### What are test containers ?

C'est une bibliothèque open source permettant de fournir des instances légères et jetables de bases de donnée etc dans un container Docker pour notamment faire des tests.

#### Document your GitHub Actions configurations


1. **Trigger Conditions** (`on`):
    - **Push Events**: Le workflow est déclenché quand on push sur main et develop.
    - **Pull Requests**: It also runs on every pull request to ensure the code is up to quality standards before merging.

2. **Job Definition** (`jobs`):
    - **Job Name**: `test-backend`
        - **Runs-On**: Le job utilise un environnement `ubuntu-22.04`.

3. **Job Steps** (`steps`):
    - **Checkout Repository** : Utilise `actions/checkout@v2.5.0` pour cloner le dépôt, rendant le code accessible dans le flux de travail.
    - **Setup JDK 17** : Utilise l'action `actions/setup-java@v3` pour installer le JDK 17 avec la distribution Amazon Corretto, qui est compatible avec de nombreuses applications Java.
    - **Build and test with Maven** : Exécute `mvn clean verify` dans le répertoire `simple-api` pour construire le projet et exécuter les tests.

Traduit avec DeepL.com (version gratuite)


#### For what purpose do we need to push docker images ?

Afin de travailler plus facilement en équipe, de déployer un système sur un serveur distant et garder un historique des versions fonctionnelles en cas de problème.

#### Document your quality gate configuration

On utilise la commande suivante pour se connecter au projet SonarCloud avec les bons identifiants pour lui permettre d'analyser et de détecter de potentielles failles et erreurs.
'run: mvn -B verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=Fabrikot_devops-livecoding -Dsonar.login=${{ secrets.SONAR_TOKEN }}' 

---------------------
--------------------

## TP N°3 ANSIBLE

## Document your inventories and base commands

L'inventory d'ansible permet de configurer le seveur host.
On différéncie deux types de configurations
1. **All Hosts Group**: Qui ici sert à définir des variables en commun pour tous les utilisateurs
2. **Children Groups**: Sous-groupes plus précis.

### Inventory Content

```yaml
all:
  vars:
    ansible_user: admin
    ansible_ssh_private_key_file: /home/fabien/ansible_id_rsa
  children:
    prod:
      hosts:
        marion.chineaud.takima.cloud
```
### Explanation of Key Components

- **`all`** : Ce groupe de premier niveau inclut tous les hôtes et peut définir des variables globales applicables à tous les groupes enfants.
  
- **`vars`** : Les variables définies ici seront appliquées à tous les hôtes de cet inventaire.
  - **`ansible_user`** : L'utilisateur SSH utilisé pour se connecter aux hôtes.
  - **`ansible_ssh_private_key_file`** : Le chemin vers la clé privée utilisée pour l'authentification SSH.

- **`children`** : Cette section catégorise les hôtes en différents environnements ou groupes. 
  - **`prod`** : Un groupe enfant représentant l'environnement de production.
    - **`hosts`** : Liste les hôtes de ce groupe. Ici, il contient `marion.chineaud.takima.cloud` car j'ai dû utiliser un serveur de backup.

## Document your playbook

## Tasks

### 1. Install Required Packages

```yaml
- name: Install required packages
  apt:
    name:
      - apt-transport-https
      - ca-certificates
      - curl
      - gnupg
      - lsb-release
      - python3-venv
    state: latest
    update_cache: yes
```
Permet d'installer les librairies nécessaires sur le serveur distant.

### 2. Add Docker’s Official GPG Key

```yaml
- name: Add Docker GPG key
  apt_key:
    url: https://download.docker.com/linux/debian/gpg
    state: present
```
Cette tâche ajoute la clé GPG nécessaire à la vérification des paquets Docker.

### 3. Set Up the Docker Stable Repository

```yaml
- name: Add Docker APT repository
  apt_repository:
    repo: "deb [arch=amd64] https://download.docker.com/linux/debian {{ ansible_facts['distribution_release'] }} stable"
    state: present
    update_cache: yes
```
Cette tâche configure le dépôt APT afin de s'assurer que la dernière version de Docker est installée.

### 4. Install Docker

```yaml
- name: Install Docker
  apt:
    name: docker-ce
    state: present
```
Permet d'installer docker.

### 5. Install Python3 and pip3

```yaml
- name: Install Python3 and pip3
  apt:
    name:
      - python3
      - python3-pip
    state: present
```
Vérifie que python est installé et installe sinon.

### 6. Create a Virtual Environment for Python Packages

```yaml
- name: Create a virtual environment for Docker SDK
  command: python3 -m venv /opt/docker_venv
  args:
    creates: /opt/docker_venv  # Only runs if this directory doesn’t exist
```
Cette tâche crée un environnement virtuel pour isoler les paquets Python.

### 7. Install Docker SDK for Python in the Virtual Environment

```yaml
- name: Install Docker SDK for Python in virtual environment
  command: /opt/docker_venv/bin/pip install docker
```
Installe docker dans l'environnement virtuel venv.

### 8. Ensure Docker is Running

```yaml
- name: Make sure Docker is running
  service:
    name: docker
    state: started
  tags: docker
```
Vérifie que docker est bien lancé.

## Document your playbook

1. Launch db

```yml
- name: Launch my db
  docker_container:
    name: my-db
    image: fabrikot/tp-devops-database
    restart_policy: always
    networks:
      - name: my-network
    volumes:
      - db-volume:/var/lib/postgresql/data
    state: started
```


2. Launch application

```yml
- name: Launch my app
  docker_container:
    name: my-api
    image: fabrikot/tp-devops-simple-api
    restart_policy: always
    networks:
      - name: my-network
    state: started
```


3. Launch proxy

```yml
- name: Launch my proxy
  docker_container:
    name: httpd
    image: fabrikot/tp-devops-front
    restart_policy: always
    ports:
      - 80:80
    networks:
      - name: my-network
    state: started
```
4. Launch front

```yml
- name: Launch my front
  docker_container:
    name: front
    image: fabrikot/front-app:1.0
    restart_policy: always
    ports:
      - 80:80
    networks:
      - name: my-network
    state: started
```
5. Create newtork (before)

```yml
- name: Create Docker network
  docker_network:
    name: my-network
    state: present
```

Merci d'avoir lu mon README ! 🚀
