# Rocket chat & key cloak integration guide

[![Develop on Okteto](https://okteto.com/develop-okteto.svg)](https://cloud.okteto.com/deploy?repository=https://github.com/nomorechokedboy/rocketchat-keycloak&branch=master)

## Prerequisite

- You should have `docker` and `docker-compose` knowledge before using this

- `Docker` and `docker-compose` installation

## Setup

- First thing first, we are going to integrate keycloak with rocket chat, thus, we need to have a rocket chat service ready

```yaml
version: '3'
services:
        rocketchat:
                container_name: rocketchat
                image: rocket.chat:latest
                restart: unless-stopped
                environment:
                        - PORT=3000
                        - MONGO_URL=/url/to/your/mongodb
                        # Reference for mongodb oplog
                        # https://forums.meteor.com/t/what-should-be-in-mongo-url-and-mongo-oplog-url-for-monglab/19356
                        # if your mongodb service is in docker-compose:
                        # mongodb://mongo:27017/local
                        # ref: https://blog.jarrousse.org/2022/04/26/using-docker-compose-in-to-deploy-rocket-chat/
                        - MONGO_OPLOG_URL=/url/to/your/mongodb/oplog
                ports:
                        - 3000:3000
```

- The code below declare a `service` name `rocketchat`, `ports` map the service port's localhost:3000 to 3000 on host

- You have to provide your url to `mongodb`. Because of some error so in the compose file, I won't deploy `mongodb` with the compose specs.

- Next, we will setup the keycloak service

```yaml
version: '3'
services:
        rocketchat:
                #...
        postgres:
                image: postgres:13.2
                restart: unless-stopped
                environment:
                        POSTGRES_DB: ${POSTGRESQL_DB}
                        POSTGRES_USER: ${POSTGRESQL_USER}
                        POSTGRES_PASSWORD: ${POSTGRESQL_PASS}
        keycloak:
                depends_on:
                        - postgres
                environment:
                        DB_VENDOR: postgres
                        DB_ADDR: postgres
                        DB_DATABASE: ${POSTGRESQL_DB}
                        DB_USER: ${POSTGRESQL_USER}
                        DB_PASSWORD: ${POSTGRESQL_PASS}
                        KEYCLOAK_ADMIN: admin
                        KEYCLOAK_ADMIN_PASSWORD: admin
                        # auto set first user for keycloak
                        KEYCLOAK_USER: admin
                        KEYCLOAK_PASSWORD: admin
                        # this env enable using https instead of http
                        # set to false if you aren't to use https
                        PROXY_ADDRESS_FORWARDING: true
                image: jboss/keycloak
                ports:
                        - '8080:8080'
                restart: unless-stopped
```

## Deploy to `okteto`

Sign in `okteto` website and then choose your github repo with this compose file and everything will be done automatically with okteto

Or you can use the button `Develop on Okteto` on [top](#rocket-chat--key-cloak-integration-guide) of the README file
