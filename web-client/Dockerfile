FROM node:23-alpine AS builder

WORKDIR /app

COPY package*.json ./

RUN yarn install

COPY . .

RUN yarn run build

FROM node:18-alpine

WORKDIR /app

RUN adduser appuser -DH

USER appuser

COPY --from=builder /app ./

EXPOSE 3000

CMD ["yarn", "start"]