# Dripstore API

Para executar este ambiente completo (API + MySQL + MariaDB + GLPI), crie um arquivo `.env` e um `docker-compose.yml` no mesmo diretório.

### 1. crie o arquivo `.env`
```env
JWT_SECRET = "ADICIONE SEU TOKEN SECRETO"
DB_HOST=localhost
DB_PORT="PORTA DO BANCO DA APLICAÇÃO"
DB_NAME="NOME DO BANCO DA APLICAÇÃO"
DB_USER="SEU USUÁRIO"  
DB_PASSWORD="SUA SENHA"
PORT="PORTA EXPOSTA NO BACKEND"
DB_HOST_DOCKER="DB HOST NO DOCKER"

#MariaDB (GLPI):
MARIADB_PASSWORD="SUA SENHA"
MARIADB_DATABASE= "NOME DO DATABASE"
MARIADB_ROOT_PASSWORD="SENHA DO USUARIO ROOT"
MARIADB_USER="NOME DE USUÁRIO"
MARIADB_PORT="PORTA DO MARIO DB"
```

### 2. crie o arquivo `docker-compose.yml`
```yaml
services:
  api:
    image: galbuquerque1012/dripstore-api:latest
    container_name: api
    ports: 
      - "${PORT:-3000}:3000"
    environment:
      DB_HOST: ${DB_HOST_DOCKER}
      DB_PORT: 3306
      DB_USER: ${DB_USER}
      DB_PASSWORD: ${DB_PASSWORD}
      DB_NAME: ${DB_NAME}
      JWT_SECRET: ${JWT_SECRET}
    depends_on:
      db:
        condition: service_healthy 
     networks:
      api-glpi:
      api-db: 
  db:
    container_name: db
    image: mysql:8.0
    ports: 
      - "${DB_PORT:-3307}:3306"
    environment:
      MYSQL_ROOT_PASSWORD: ${DB_PASSWORD}
      MYSQL_PASSWORD: ${DB_PASSWORD}
      MYSQL_DATABASE: ${DB_NAME}
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      timeout: 20s
      retries: 10
      interval: 10s
      start_period: 10s
    volumes:
      - db:/var/lib/mysql
    networks:
      api-db:
  mariadb:
    image: mariadb:10.7
    container_name: mariadb
    hostname: mariadb
    ports: 
      - "${MARIADB_PORT:-3007}:3306"
    environment:
      MARIADB_ROOT_PASSWORD: ${MARIADB_ROOT_PASSWORD}
      MARIADB_DATABASE: ${MARIADB_DATABASE}
      MARIADB_USER: ${MARIADB_USER}
      MARIADB_PASSWORD: ${MARIADB_PASSWORD}
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-u", "root", "-p${MARIADB_ROOT_PASSWORD}"]
      timeout: 5s
      retries: 10
      interval: 10s
      start_period: 15s
    volumes:
      - mariadb:/var/lib/mysql
    networks:
      glpi-mariadb:
  glpi:
    image: diouxx/glpi
    container_name: glpi
    hostname: glpi
    ports:
      - "8080:80"
    depends_on:
      mariadb:
        condition: service_healthy
    volumes:
      - glpi_data:/var/www/html/glpi
    networks:
      glpi-mariadb:
      api-glpi:
volumes:
  db:
  mariadb:
  glpi_data:
networks:
  api-glpi:
  api-db:
  glpi-mariadb:
```

### 3. Como Executar
```bash
docker compose up -d
```
