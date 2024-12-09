# Utilisation de l'image Node.js pour le build
FROM node:23 AS builder

WORKDIR /app

# Copie des fichiers de package et installation des dépendances
COPY package*.json ./
RUN yarn install

# Copie du reste du code source
COPY . .

# Construction de la documentation
RUN yarn build

# Utilisation de Nginx pour servir la documentation statique
FROM nginx:alpine

# Copie des fichiers de build vers le dossier de Nginx
COPY --from=builder /app/build /usr/share/nginx/html

# Exposer le port 80 pour accéder à la documentation
EXPOSE 80

# Démarrage de Nginx
CMD ["nginx", "-g", "daemon off;"]
