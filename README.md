# Keycloak Docker setup with NGINX Proxy and Letsencrypt

A simple Keycloak setup using nginx reverkeycloak-docker-sslse proxy, docker gen and letsencrypt for generating SSL certs.
Keycloak is a powerful Open Source Identity and Access Management application.

### Reference
* NGINX Proxy Docker image: https://github.com/jwilder/nginx-proxy
* Letsencrypt NGINX Docker Companion: https://github.com/JrCs/docker-letsencrypt-nginx-proxy-companion

## Guide
Copy sample.env to .env and adjust the environment variables.
Then ```docker-compose up```

Note: ssl option in JDBC_PARAMS is set to false, as the nginx proxy will handle SSL.
```
DB_DATABASE=keycloak_db
DB_USER=keycloak_db_user
DB_PASSWORD=Passw0rd!
KEYCLOAK_HOSTNAME=example.com
KEYCLOAK_HTTP_PORT=8080
KEYCLOAK_USER=admin
KEYCLOAK_PASSWORD=Passw0rd!
JDBC_PARAMS="ssl=false" 
VIRTUAL_HOST=example.com
VIRTUAL_PORT=8080
PROXY_ADDRESS_FORWARDING=true
LETSENCRYPT_HOST=example.com
LETSENCRYPT_EMAIL=luke.skywalker@example.com
```

## docker-compose.yml

```
version: '3'

services:
  nginx-proxy:
    image: jwilder/nginx-proxy:alpine
    container_name: nginx-proxy
    restart: always
    labels:
      com.github.jrcs.letsencrypt_nginx_proxy_companion.nginx_proxy: 'true'
    ports:
      - 80:80
      - 443:443
    volumes:
      - ./nginx/data/certs:/etc/nginx/certs:ro
      - ./nginx/data/conf.d:/etc/nginx/conf.d
      - ./nginx/data/vhost.d:/etc/nginx/vhost.d
      - ./nginx/data/html:/usr/share/nginx/html
      - /var/run/docker.sock:/tmp/docker.sock:ro
    networks:
      - webproxy

  letsencrypt:
    image: jrcs/letsencrypt-nginx-proxy-companion
    container_name: letsencrypt
    volumes:
      - ./nginx/data/vhost.d:/etc/nginx/vhost.d
      - ./nginx/data/certs:/etc/nginx/certs:rw
      - ./nginx/data/html:/usr/share/nginx/html
      - /var/run/docker.sock:/var/run/docker.sock:ro
    depends_on:
      - nginx-proxy
    networks:
      - webproxy

  keycloak:
    image: jboss/keycloak
    container_name: keycloak
    restart: always
    environment: 
      DB_DATABASE: ${DB_DATABASE}
      DB_USER: ${DB_USER}
      DB_PASSWORD: ${DB_PASSWORD}
      JDBC_PARAMS: ${JDBC_PARAMS}
      KEYCLOAK_HOSTNAME: ${KEYCLOAK_HOSTNAME}
      KEYCLOAK_HTTP_PORT: ${KEYCLOAK_HTTP_PORT}
      KEYCLOAK_USER: ${KEYCLOAK_USER}
      KEYCLOAK_PASSWORD: ${KEYCLOAK_PASSWORD}
      VIRTUAL_HOST: ${VIRTUAL_HOST}
      VIRTUAL_PORT: ${VIRTUAL_PORT}
      PROXY_ADDRESS_FORWARDING: ${PROXY_ADDRESS_FORWARDING}
      LETSENCRYPT_HOST: ${LETSENCRYPT_HOST}
      LETSENCRYPT_EMAIL: ${LETSENCRYPT_EMAIL}
    depends_on:
      - postgres
    networks:
      - webproxy

  postgres:
    image: postgres
    container_name: postgres
    restart: always
    environment:
      POSTGRES_DB: ${DB_DATABASE}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - data:/var/lib/postgresql/data
    networks:
      - webproxy  
   
networks:
  webproxy:

volumes:
  data:
```

