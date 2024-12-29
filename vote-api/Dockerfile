# Étape 1 : Construction de l'application
FROM golang:1.23 AS builder

WORKDIR /app

# Copie des fichiers nécessaires pour la construction
COPY go.mod go.sum ./
RUN go mod download

COPY . .

# Construction de l'application Go
RUN go build -o vote-api

# Étape 2 : Création de l'image finale
FROM alpine:latest

# Installation des certificats CA pour les connexions sécurisées
RUN apk --no-cache add ca-certificates

WORKDIR /root/

# Copie du binaire de l'étape précédente
COPY --from=builder /app/vote-api .

# Vérification de la présence du fichier (utile pour le débogage)
RUN ls -l /root/

# Attribution des permissions d'exécution
RUN chmod +x /root/vote-api

# Exposition du port de l'API
EXPOSE 8080

# Démarrage de l'application
CMD ["./vote-api"]
docker run --rm -p 8080:8080 -p 5432:5432 img:latest
