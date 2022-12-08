<!-- Armar un readme.MD con 'code' sobre como dockerizar una app de nest -->
# Dockerizar una app de NestJS

## Modificar el nest-cli.json

```json
{
  "$schema": "https://json.schemastore.org/nest-cli",
  "collection": "@nestjs/schematics",
  "sourceRoot": "src",
  "compilerOptions": {
    "webpack": true
  }
}
```

## Crear el Dockerfile de desarrollo

```Dockerfile
# Dockerfile.dev
FROM node:18

# Create app directory
WORKDIR /usr/src/app

# Entrypoint npm run start:dev
ENTRYPOINT ["npm", "run", "start:dev"]
```

## Crear el Dockerfile de producción

```Dockerfile
# Dockerfile
# build image
FROM node:18 as install

# Create app directory
WORKDIR /usr/src/install

# Install app dependencies
COPY package*.json ./
# change network timeout
RUN npm config set network-timeout 60000
# Install dependencies
RUN npm install

# Compile app
FROM node:18 as build

# Create app directory
WORKDIR /usr/src/build

# Copy install dependencies
COPY --from=install /usr/src/install/node_modules ./node_modules

# Copy app source
COPY . .

# Build app
RUN npm run build
# change network timeout
RUN npm config set network-timeout 60000
# Install dependencies prod
RUN npm install --only=prod

# Deploy app
FROM node:18-alpine as deploy

# Create app directory
WORKDIR /usr/src/app

# Copy build dependencies
COPY --from=build /usr/src/build/node_modules ./node_modules
COPY --from=build /usr/src/build/dist/main.js index.js

# # Expose port
# EXPOSE 3000

# Entrypoint node .
ENTRYPOINT ["node", "."]

```

## Crear el docker-compose.yml

```docker-compose.yml
# docker-compose.yml
version: '3.8'

services:
  app:
    image: 'nestjs-docker:local'
    container_name: 'nestjs-docker'
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - '3000:3000'
    volumes:
      - .:/usr/src/app
    # environment:
      # - NODE_ENV=development
    # command: npm run start:dev
    command: docker build . -t nestjs-docker:latest && docker run -d -p 3000:3000 --name nestjs-docker-container nestjs-docker
```



## Buildear la imagen 

```bash
    docker build . -t nestjs-docker:latest
```

## Levantar la imagen 

```bash
    docker compose up
```

##### Si no funciona el `command` del `docker-compose.yml`

```bash
    docker run -d -p 3000:3000 --name nestjs-docker-container nestjs-docker
```