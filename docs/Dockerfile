# Étape de construction
FROM node:23-alpine AS builder

# Création du répertoire de travail
WORKDIR /app

# Copie des fichiers de dépendances
COPY package*.json ./

# Installation des dépendances
RUN npm install

# Copie du code source
COPY . .

# Construction de la documentation
RUN npm run build

# Étape de production
FROM nginx:alpine

# Copie des fichiers construits dans le répertoire de nginx
COPY --from=builder /app/build /usr/share/nginx/html

# Exposition du port
EXPOSE 80

# Commande pour démarrer nginx
CMD ["nginx", "-g", "daemon off;"]