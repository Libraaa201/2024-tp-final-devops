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