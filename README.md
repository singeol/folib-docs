## Быстрый запуск в docker compose

### Создадим папки для folib
```sh
mkdir -p ~/folib-stack/data/mysql
mkdir -p ~/folib-stack/data/folib/folib-conf
mkdir -p ~/folib-stack/data/folib/folib-data
mkdir -p ~/folib-stack/data/folib/tmp
cd ~/folib-stack
```

### Откроем файл в текстовом редакторе, например vim
```sh
vim docker-compose.yaml
```

### Минимальный docker compose
```docker
version: "3.9"

services:
  mysql:
    image: mysql:8.0
    container_name: folib-mysql
    command: >
      --character-set-server=utf8mb4
      --collation-server=utf8mb4_general_ci
      --default-authentication-plugin=mysql_native_password
    environment:
      MYSQL_ROOT_PASSWORD: "folib@v587"
      MYSQL_DATABASE: "folib"
    volumes:
      - ./data/mysql:/var/lib/mysql
    ports:
      - "3306:3306"
    healthcheck:
      # ждём готовности MySQL
      test: ["CMD", "mysqladmin", "ping", "-h", "127.0.0.1", "-uroot", "-pfolib@v587"]
      interval: 10s
      timeout: 5s
      retries: 10
      start_period: 30s
    restart: unless-stopped

  folib:
    image: ghcr.io/singeol/folib:latest
    container_name: folib-server
    depends_on:
      mysql:
        condition: service_healthy
    environment:
      FOLIB_PORT: "38080"
      FOLIB_JVM_XMX: "4096m"
      FOLIB_JVM_XMS: "4096m"
      FOLIB_JVM_XSS: "512k"
      FOLIB_MYSQL_HOST: "mysql"
      FOLIB_MYSQL_PORT: "3306"
      FOLIB_MYSQL_DB: "folib"
      FOLIB_MYSQL_USER: "root"
      FOLIB_MYSQL_PASSWORD: "folib@v587"
      FOLIB_ARTIFACT_UPLOAD_RESTRICTIONS: "true"
    volumes:
      - ./data/folib/folib-conf:/opt/folib/folib-3.0-SNAPSHOT/etc/conf
      - ./data/folib/folib-data:/opt/folib/folib-data
      - ./data/folib/folib-data/logs:/opt/folib/folib-data/logs
      - ./data/folib/tmp:/opt/folib/folib-3.0-SNAPSHOT/tmp
    ports:
      - "38080:38080"
      - "7010:7010"
      - "7011:7011"
      - "7199:7199"
      - "49142:49142"
      - "8182:8182"
    restart: unless-stopped
```

### Запустим docker compose с folib
```sh
docker compose up -d
```
