# Étape de construction
FROM golang:1.20 AS builder

# Création du répertoire de travail
WORKDIR /app

# Copie des fichiers Go
COPY . .

# Installation des dépendances et compilation
RUN go mod tidy
RUN go build -o vote-api .

# Étape de production
FROM alpine:latest

# Copie du binaire depuis l'étape builder
COPY --from=builder /app/vote-api /usr/local/bin/vote-api

# Exposition du port
EXPOSE 4000

# Commande pour démarrer l'application
CMD ["vote-api"]