# Documentation du Projet 

## Vue d'ensemble

Ce document présente les processus et résultats du projet DevOps, y compris la Dockerisation, les pipelines CI/CD et les stratégies de déploiement. L'objectif est de fournir une vue d'ensemble claire de l'architecture et des pratiques de développement pour le projet, afin que de nouveaux utilisateurs puissent rapidement comprendre comment y contribuer.

## Objectifs du Projet

Le projet vise à accomplir les objectifs suivants :

> Créer des images Docker pour chaque composant du projet (web-client, vote-api, docs).

> Créer un fichier Docker Compose pour orchestrer les services et les connexions entre eux.

> Créer un pipeline CI/CD avec GitHub Actions pour automatiser les processus de tests, de build, et de déploiement.

> Déployer l'application sur plusieurs plateformes d'hébergement pour garantir la scalabilité et la disponibilité.

## Architecture du Projet

Ce projet est structuré autour de plusieurs composants principaux :

1. **Documentation** : Un site statique généré avec Docusaurus, centralisant toutes les informations relatives au projet, à sa gestion et à sa contribution. Déployé sur Netlify.

2. **Web-client** : L'interface utilisateur de l'application, développée avec Next.js. Elle permet aux utilisateurs d'interagir avec l'application. Déployée sur Render.

3. **Vote-API** : L'API principale du projet, écrite en Go. Elle récupère des données de l'API TVMaze et les met à disposition pour l'interface utilisateur. Déployée sur Render.

4. **Database** : La base de données qui stocke les informations. Déployée sur Render.
    
## Concepts Clés

- Dockerfile : Un fichier de configuration utilisé pour automatiser la création d'images Docker. Il contient les instructions nécessaires pour configurer l'environnement, installer les dépendances et définir le comportement de l'application dans le conteneur.

- Docker Compose : Un outil permettant de définir et de gérer plusieurs conteneurs Docker. Il est utilisé pour orchestrer les services du projet (web-client, vote-api, base de données PostgreSQL).

- Pipeline CI/CD avec GitHub Actions : Un ensemble de processus automatisés permettant de valider, tester, construire, et déployer l'application sur des environnements de production par exemple. Il facilite l'intégration continue (CI) et le déploiement continu (CD) du code.

## Workflow Git

### Stratégie de Branches
Dans notre cas, nous n'avons pas eu le réflexe d'organiser correctement nos branches. Nous avons modifié la branche 'main' directement une fois la modification réalisée et fonctionnelle en local.

Un projet bien orchestré doit suivre suit une stratégie de branches GitFlow pour organiser le développement du code. Voici les branches principales :

- **main** : Branche stable contenant le code prêt pour la production.

- **develop** : Branche d'intégration, utilisée pour tester les nouvelles fonctionnalités et corrections de bugs avant qu'elles ne soient fusionnées dans main.

- **feature** : Branche pour le développement de nouvelles fonctionnalités.

- **hotfix** : Branche pour les corrections urgentes de production
    
    

### Pull requests

Pour éviter d'utiliser énormément de pull request lors de la conception des GitHub Actions, nous avons utilisé **Act**, permettant de les exécuter en local.

La réalisation de ces scripts d'intégration continue (CI) a permis ensuite de tester et vérifier les codes avant leur fusion. Cela a permit de réaliser une revue automatique du fonctionnement du code. 

### Rollbacks

De même que pour l'organisation des branches, nous n'avons pas utilisé de tags. Or, les rollbacks peuvent être gérés via des tags Git. De ce fait, si une version déployée pose problème, on revient à la dernière version stable en redéployant l’image Docker correspondante.

### Processus de hotfix 

Pour créer un hotfix, suivre ces étapes :

1. Créez une branche à partir de main.
2. Appliquez la correction nécessaire et effectuez les tests.
3. Fusionnez la branche dans main et redéployez.

## Docker

Une image Docker a été réalisé pour chaque partie du projet, c'est à dire pour le front-end (web-client), le back-end (vote-api), et la documentation (docs).

### Dockerfile Docs

Construit avec Node.js et servie par Nginx, cette image permet de déployer une application web contenant toute la documentation du projet.

```bash
FROM node:23-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM nginx:alpine
RUN adduser -DH appuser
USER appuser
COPY --from=builder /app/build /usr/share/nginx/html

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

On peut classer l'image en 3 parties : 

- **Build**
    - Utilise une image Node.js légère (Alpine)
    - Définit le répertoire de travail et copie les fichiers nécessaires pour installer les dépendances (`package.json`, `package-lock.json`).
    - Exécute `npm install` pour installer les dépendances, puis copie tout le code source et exécute `npm run build` pour créer l'application dans le répertoire `build`.

- **Production avec Nginx**
    - Passe à une nouvelle image de base Nginx, également légère (Alpine).
    - Crée un nouvel utilisateur (non-root) pour des raisons de sécurité, puis change le contexte d'exécution pour cet utilisateur.
    - Copie les fichiers construits de l'étape précédente (du répertoire `build` à `/usr/share/nginx/html`), où Nginx cherche les fichiers à servir.

- **Configuration et exécution**
    - Expose le port 80 pour que Nginx soit accessible.
    - Définit la commande par défaut pour démarrer Nginx en gardant le processus au premier plan, ce qui empêche le conteneur de s'arrêter.

### Dockerfile Web-Client

Cette image permet de déployer le front-end 'web-client' du projet.

```bash
FROM node:23-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN yarn install
COPY . .
RUN yarn build

FROM node:18-alpine
WORKDIR /app
RUN adduser -DH appuser
USER appuser

COPY --from=builder /app ./
EXPOSE 3000
CMD ["yarn", "start"]
```

L'image est découpée en 3 parties :

- **Build**
    - Utilise une image Node.js
    - Définit le répertoire de travail à `/app`
    - Exécute `yarn install` pour installer les dépendances, puis copie tout le code source et exécute `yarn build`.

- **Production**
    - Passe à une nouvelle image Node.js.
    - Définit de nouveau le répertoire de travail à `/app`, crée un nouvel utilisateur non-root (`appuser`), puis change le contexte d'exécution pour cet utilisateur.

- **Déploiement et exécution**
    - Copie les fichiers construits de l'étape précédente dans le répertoire de travail de cette image.
    - Expose le port 3000.
    - Définit la commande par défaut pour démarrer l'application avec `yarn start`.



### Dockerfile Vote-API

Ce Dockerfile est conçu pour construire et déployer le back-end 'vote-api' codé en Go.

```bash
FROM golang:1.23 AS builder

WORKDIR /app

COPY go.mod ./
COPY go.sum ./
RUN go mod tidy

COPY . .

RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -a -installsuffix cgo -o vote-api .

FROM alpine:latest

COPY --from=builder /app/vote-api /usr/local/bin/vote-api

RUN adduser appuser -DH

USER appuser

ENV PG_URL="postgres://user:password@host.docker.internal:5432/postgres?sslmode=disable"
EXPOSE 8080

CMD ["vote-api"]
```

L'image est découpée en 3 parties :

- **Build**
    - Utilise l'image Golang 1.23 comme environnement de construction.
    - Configure le répertoire de travail à `/app`, où il gère les dépendances à l'aide de `go.mod` et `go.sum`.
    - Exécute `go mod tidy` pour installer les dépendances.
    - Copie tout le code source et compile l'application pour produire un binaire exécutable.

- **Production**
    - Change pour une image Alpine légère.
    - Copie le binaire `vote-api` de l'étape de construction vers le répertoire `/usr/local/bin` de l'image Alpine.

- **Configuration et exécution**
    - Crée un utilisateur non-root empêchant l'exécution en root.
    - Définit une variable d'environnement pour la connexion à une base de données PostgreSQL.
    - Expose le port 8080 pour permettre l'accès à l'application.
    - Spécifie la commande par défaut pour exécuter le binaire `vote-api` lorsque le conteneur démarre.

### Docker Compose

Le fichier `docker-compose.yaml` orchestre les services du projet et configure les connexions réseau.

```bash
version: '3.9'
services:
  db:
    image: postgres:latest
    container_name: devopsfinal
    environment:
      POSTGRES_DB: postgres
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"
    networks:
      - vote-api-network

  vote-api:
    build:
      context: vote-api
    ports:
      - "8080:8080"
    environment:
      PG_URL: "postgres://user:password@db:5432/postgres?sslmode=disable"
    depends_on:
      - db
    networks:
      - vote-api-network

  web-client:
    build:
      context: web-client
    ports:
      - "3000:3000"
    environment:
      VOTE_API_BASE_URL: "http://vote-api:8080"
    networks:
      - vote-api-network

networks:
  vote-api-network:
```

Le fichier comporte plusieurs services.

1. **db**
> Utilise l'image PostgreSQL la plus récente pour créer un conteneur de base de données.

> Définit des variables d'environnement pour initialiser la base de données, l'utilisateur et le mot de passe.

> Expose le port 5432 pour permettre les connexions externes à la base de données.
S'inscrit dans le réseau `vote-api-network` pour faciliter la communication avec d'autres services.

2. **vote-api**

> Construit l'API à partir du répertoire vote-api.

> Expose le port 8080 pour permettre l'accès à l'API depuis l'extérieur.

> Définit une variable d'environnement `PG_URL` pour connecter l'API à la base de données PostgreSQL en utilisant le service `db`.

> Utilise la directive `depends_on` pour garantir que le service `db` est démarré avant l'API.

> Fait également partie du réseau `vote-api-network`.

3. **web-client**

> Construit le client web à partir du répertoire web-client.

> Expose le port 3000 pour que l'application soit accessible.

> Définit une variable d'environnement ``VOTE_API_BASE_URL`` pour indiquer à l'application client l'URL de l'API de vote.

> Comme les autres services, il est connecté à ``vote-api-network.``

4. **networks**

> Définit un réseau personnalisé ``vote-api-network`` pour permettre aux services de communiquer entre eux.

### Pipelines CI/CD

##### CI Front-end (``ci-front.yaml``)

Ce pipeline valide le code du web-client en effectuant :

1. La configuration de Node.js.
2. L'installation des dépendances avec Yarn.
3. L'exécution des tests unitaires.
4. Le processus de build.

**NOTE :** Dans le fichier de test ``vitest.config.mts``, nous avons modifier le code afin d'exclure ``**/e2e/**.ts`` et ``**/tests-examples/**.ts``, car ceux-ci comportaient des erreurs lors du lancement de la CI concernant le code web-client.

```bash
name: CI Front
 
permissions:
  actions: read
  contents: read

on:
  push:
    branches:
      - main
  pull_request:

defaults:
  run:
    working-directory: web-client

jobs:
  tests:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Install NodeJS
        uses: actions/setup-node@v4
        with:
          node-version: 22

      - name: Install Yarn
        run: npm install -g yarn

      - name: Install NodeJS
        uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: 'yarn'
          cache-dependency-path: './web-client/yarn.lock'

      - name: Install dependencies
        run: yarn install --frozen-lockfile

      - name: Run tests
        run: yarn test

  build:
    runs-on: ubuntu-latest
    needs: tests
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Install NodeJS
        uses: actions/setup-node@v4
        with:
          node-version: 22

      - name: Install Yarn
        run: npm install -g yarn

      - name: Install NodeJS
        uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: 'yarn'
          cache-dependency-path: './web-client/yarn.lock'

      - name: Install dependencies
        run: yarn install --frozen-lockfile

      - name: Build project
        run: yarn build
```

#### CI Back-end (``ci-back.yaml``)

Ce pipeline valide le code du vote-api en effectuant :

1. La configuration de Go.
2. La gestion des dépendances.
3. L'exécution des tests unitaires.
4. Le processus de build.

```bash
name: CI Back
 
permissions:
  actions: read
  contents: read

on:
  push:
    branches:
      - main
  pull_request:

defaults:
  run:
    working-directory: vote-api

jobs:
  tests:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Go
        uses: actions/setup-go@v5
        with:
          go-version: '1.21.x'

      - name: Install dependencies
        run: go get .

      - name: Run tests
        run: go test -v ./...

  build:
    runs-on: ubuntu-latest
    needs: tests
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Go
        uses: actions/setup-go@v5
        with:
          go-version: '1.21.x'

      - name: Install dependencies
        run: go get .

      - name: Build project
        run: go build .
```

#### CD Front-end (``cd-front.yaml``)

Ce pipeline automatise le déploiement du web-client vers Docker Hub et Render, à condition que le CI Front-end soit validé.

```bash
name: CD Front

permissions:
  actions: read
  contents: read

on:
  workflow_run:
    workflows: ["CI Front"]
    branches: [main]
    types: 
      - completed

defaults:
  run:
    working-directory: web-client

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: ./web-client
          file: ./web-client/Dockerfile
          push: true
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/web-client:latest

      - name: Re-deploy Docker container
        run: curl ${{ secrets.RENDER_HOOK_FRONT }}        
```
**NOTE** : 
- Les variables de connexion tels que ``secrets.DOCKERHUB_*`` et ``secrets.RENDER_HOOK_FRONT`` sont insérés en tant que variables secrètes sur GitHub.

- ``secrets.RENDER_HOOK_FRONT`` est une URL accédant à l'API de Render, lui indiquant de récupérer l'image du front-end présente sur Docker afin de se mettre à jour.

#### CD Back-end (``cd-back.yaml``)

Ce pipeline automatise le déploiement de vote-api vers Docker Hub et Render, à condition que le CI back-end soit validé.

```bash
name: CD Back

permissions:
  actions: read
  contents: read

on:
  workflow_run:
    workflows: ["CI Back"]
    branches: [main]
    types: 
      - completed

defaults:
  run:
    working-directory: vote-api

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: ./vote-api
          file: ./vote-api/Dockerfile
          push: true
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/vote-api:latest

      - name: Re-deploy Docker container
        run: curl ${{ secrets.RENDER_HOOK_BACK }}
```

**NOTE** : 
- Les variables de connexion tels que ``secrets.DOCKERHUB_*`` et ``secrets.RENDER_HOOK_BACK`` sont insérés en tant que variables secrètes sur GitHub.

- ``secrets.RENDER_HOOK_BACK`` est une URL accédant à l'API de Render, lui indiquant de récupérer l'image du back-end présente sur Docker afin de se mettre à jour.

## Déploiement

Web-Client et Vote-API sont déployées sur Render, et la documentation sur Netlify.

## URL des dépôts

- Render    
    > Lien Render web-client : https://web-client-n1o3.onrender.com/
    
    > Lien Render vote-api : https://vote-api-zcn1.onrender.com/

- Netlify    
    > Lien Netlify Docs : https://ineedmoretea-docs.netlify.app/docs/intro/

- Docker
    > Dépôt Docker web-client : https://hub.docker.com/repository/docker/kylianrolin/web-client/general
    
    > Dépôt Docker vote-api : https://hub.docker.com/repository/docker/kylianrolin/vote-api/general
