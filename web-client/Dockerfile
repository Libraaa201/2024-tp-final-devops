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

# Construction de l'application
RUN npm run build

# Étape de production
FROM node:18-alpine

# Création du répertoire de travail
WORKDIR /app

# Copie des fichiers construits depuis l'étape builder
COPY --from=builder /app ./

# Exposition du port
EXPOSE 3000

# Commande pour démarrer l'application
CMD ["npm", "start"]