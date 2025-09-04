# portainer-deploy-stack-action

Deploy your services to [Docker Swarm](https://docs.docker.com/engine/swarm/) cluster via [Portainer](https://www.portainer.io). 

## Features
 - create stack if not exists, update if already exists
 - grant access to spicified teams
 - works via [Portainer API](https://documentation.portainer.io/archive/1.23.2/API/)

## Usage

```yaml
name: CI

on:
  push:
    branches:
      - main 

jobs:
  deploy:
    name: Deploy
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: telemetr-me/portainer-deploy-stack-action
        with:
          # url of Poratainer instance
          portainer-url: ${{ secrets.PORTAINER_URL }}

          # portainer auth
          portainer-username: ${{ secrets.PORTAINER_USERNAME }}
          portainer-password: ${{ secrets.PORTAINER_PASSWORD }}
          
          # internal portainer cluster id
          portainer-endpoint: 1
          
          # stack name
          stack-name: whoami

          # docker stack file location
          stack-file: .github/stack/staging.yml
          
          # vars to substitute in stack
          stack-vars: |
            DOMAIN: whoami.${{ secrets.DOMAIN }}

          # grant access for specified teams
          teams: Microservices, Telemetr Family
```

### Локальная разработка
Для запуска локально необходимо установить nodejs нужной версии, на данный момент это 16я версия, сделать можно вот так:

```bash
    curl -sL https://deb.nodesource.com/setup_16.x | sudo -E bash -
    sudo apt install -y nodejs
```
Либо можно использовать docker, для этого можно создать docker-compose.yml следующего содержания:
```bash
services:
  app:
    image: node:16-bullseye
    container_name: my-node-app
    working_dir: /app
    volumes:
      - ./:/app
    stdin_open: true
    tty: true
    command: bash
```
Затем для сборки проекта необходимо выполнить команды из `.github/workflows/ci.yml`, а именно:
```bash
npm ci
npm run format-check
npm run lint
npm run build
npm run package
```
После этого должны будут обновиться файлы в `dist/`.
Команды, которые запускаются после `npm run `, например `lint` описаны в `package.json`.

На данный момент, при запуске `npm run package` может возникнуть ошибка:
`HookWebpackError: ENOENT: no such file or directory, open '/package.json'`

Эта ошибка возникает при запуске "подкоманды" т.е.:
`ncc build --source-map --license licenses.txt`

Сам `ncc` ожидает, что файл `package.json` находится либо в корне, либо в той дирректории откуда запускается команда.
Но т.к. сам исполняемый файл подтягивается как зависимость и фактически лежит в `./node_modules/.bin/ncc` то получатся и `package.json` должен лежать рядом.

Для решения этой проблемы можно установить `ncc` глобально:
`npm install -g @vercel/ncc`

И после этого уже компилятор отрабатывает корректно:
```bash
root@5dc44e499785:/app# ncc version
0.38.3
root@5dc44e499785:/app# ncc build --source-map --license licenses.txt
ncc: Version 0.38.3
ncc: Compiling file index.js into CJS
  39kB  dist/licenses.txt
  40kB  dist/sourcemap-register.js
1450kB  dist/index.js
1531kB  dist/index.js.map
1531kB  dist/index.js.map
3060kB  [2079ms] - ncc 0.38.3
root@5dc44e499785:/app# 
```
Если компилятор не знает "что ему компилировать", то это можно указать явно:
```bash
ncc build /app/lib/main.js --source-map --license licenses.txt
```






