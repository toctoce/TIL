# Docker로 배포 관리하기

## 0. 기존 배포 방식

기존에 동작하던 프로세스는 다음과 같다.

- Frontend daemon
- Backend daemon
- PostgreSQL Docker container
- Judge Worker Docker container

수정 사항을 배포하려면 DB 마이그레이션, 프론트엔드 중지, 빌드 및 배포, 백엔드 중지, 빌드 및 배포, 채점 서버 중지, 빌드, 실행 과정을 거쳐야 했다. Docker를 도입해 이 배포 과정을 간단하게 만들어 보자.

## 1. Backend

`Dockerfile` 생성

```dockerfile
FROM eclipse-temurin:21-jdk-alpine AS builder

WORKDIR /app

COPY gradlew .
COPY gradle gradle
COPY build.gradle settings.gradle ./

RUN chmod +x gradlew
RUN ./gradlew dependencies --no-daemon

COPY src src

RUN ./gradlew clean build --no-daemon

RUN cp "$(find build/libs -maxdepth 1 -type f -name '*.jar' ! -name '*-plain.jar')" /app/app.jar

FROM eclipse-temurin:21-jre-alpine AS runtime

WORKDIR /app

RUN addgroup -S app && adduser -S app -G app

COPY --from=builder --chown=app:app /app/app.jar /app/app.jar

USER app

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "/app/app.jar"]
```

`Dockerfile`을 생성한 후 이미지를 빌드하고 컨테이너를 실행해야 한다.

빌드

```bash
docker build -t ps-backend ./backend
```

실행

```bash
docker run -d \
    --name ps-backend \
    --restart unless-stopped \
    -p 127.0.0.1:8080:8080 \
    --env-file /etc/ps-platform/backend.env \
    ps-backend
```

> `env` 파일에 접근할 권한이 없다면 `sudo`로 명령어를 실행하면 된다.

> 문제 상황: Backend가 DB에 접근하지 못함
>
> 1. `env` 파일의 `DB_HOST`가 `127.0.0.1`이라면 백엔드 컨테이너는 자신의 컨테이너에서 실행 중인 DB를 찾는다. `env` 파일의 `DB_HOST`를 DB 컨테이너 이름으로 변경하면 해결된다.
> 2. 두 컨테이너의 네트워크가 다르다면 같게 설정해야 한다. `docker run` 옵션에 `--network infra_default`를 추가한다.

실행한 컨테이너를 중지하고 삭제하려면 다음 명령어를 사용한다.

```bash
docker stop ps-backend
docker rm ps-backend
```

## 2. Frontend

```text
  인터넷
     ↓ 80/443
  web 컨테이너(Nginx + React dist)
     ├─ /*      → React 정적 파일 직접 제공
     └─ /api/* → backend:8080

  backend
     └─ postgres:5432

  judge-worker
     ├─ backend:8080
     └─ Docker socket → 채점 컨테이너 실행
```

프론트엔드 폴더에 `nginx.conf` 파일을 생성한다.

```nginx
server {
    listen 80;
    server_name algorithm.zzanggyu.com;
    
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name algorithm.zzanggyu.com;

    ssl_certificate /etc/letsencrypt/live/algorithm.zzanggyu.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/algorithm.zzanggyu.com/privkey.pem;
    
    root /usr/share/nginx/html;
    index index.html;
    
    location /api/ {
        proxy_pass http://ps-backend:8080; # 백엔드 컨테이너 이름
        
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

프론트엔드 `Dockerfile` 작성

```dockerfile
FROM node:22-alpine AS builder

WORKDIR /app

COPY package.json package-lock.json ./

RUN npm ci

COPY . .

RUN npm run build

FROM nginx:1.28.3-alpine AS runtime

COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=builder /app/dist /usr/share/nginx/html

EXPOSE 80
```

`.dockerignore` 파일 작성

```text
node_modules
dist
.git
.gitignore
.DS_Store
*.log
```

빌드

```bash
docker build -t ps-web ./frontend
```

테스트를 위해 사용하지 않는 8088번 포트에서 임시로 실행한다.

```bash
docker run -d \
    --name ps-web \
    --restart unless-stopped \
    --network infra_default \
    -p 8088:80 \
    ps-web
```

> 테스트를 위해 8088번 포트를 외부에 임시로 공개한다.
>
> AWS 인바운드 규칙에서 8088번 포트를 열면 `http://<domain>:8088`로 접속할 수 있다. 테스트가 끝나면 인바운드 규칙을 제거한다.
>
> ![프론트엔드 컨테이너 실행 확인](<Docker로 배포 관리하기 1.png>)
>
> ![프론트엔드 접속 확인](<Docker로 배포 관리하기 2.png>)

이제 실행하면 된다. 기존 Nginx를 종료하고 Docker로 실행해야 하는데, 그 사이에 몇 초간 서비스가 중단될 수 있으므로 이 점에 유의하자.

```bash
sudo systemctl stop nginx

docker run -d \
    --name ps-web \
    --restart unless-stopped \
    --network infra_default \
    -p 80:80 \
    -p 443:443 \
    -v /etc/letsencrypt:/etc/letsencrypt:ro \
    ps-web
```

> 현재 인증서 재발급 방식은 다음과 같다.
>
> ```text
> Certbot이 호스트 디렉터리에 파일 생성
>            ↓
> 컨테이너 Nginx는 해당 디렉터리를 모름
>            ↓
> Let’s Encrypt가 파일을 찾지 못함
>            ↓
> 인증서 갱신 실패
> ```
>
> 따라서 컨테이너 Nginx 방식에 맞게 수정해야 한다. 자세한 내용은 별도 게시글에서 작성하겠다.

## 3. Judge Worker (채점 서버)

```dockerfile
FROM golang:1.26.4-alpine AS builder

WORKDIR /app

COPY go.mod ./
RUN go mod download

COPY . .

RUN CGO_ENABLED=0 GOOS=linux \
    go build \
    -trimpath \
    -ldflags="-s -w" \
    -o /out/judge-worker \
    ./cmd/judge-worker

FROM alpine:3.23 AS runtime

RUN apk add --no-cache \
    ca-certificates \
    docker-cli

WORKDIR /app

COPY --from=builder /out/judge-worker ./judge-worker

ENTRYPOINT ["./judge-worker"]
```

`.dockerignore` 작성

```text
.git
.gitignore
.gocache
build
*.log
.env
.env.*
!.env.example
```

이미지 빌드

```bash
docker build -t ps-judge-worker ./judge-worker
```

환경 변수 파일의 백엔드 주소 수정

```env
# BACKEND_BASE_URL=http://127.0.0.1:8080
BACKEND_BASE_URL=http://ps-backend:8080
```

채점 코드를 임시로 저장할 폴더 `/var/lib/ps-platform/judge-work`를 생성한다.

실행

```bash
sudo docker run -d \
    --name ps-judge-worker \
    --restart unless-stopped \
    --network infra_default \
    --env-file /etc/ps-platform/judge-worker.env \
    -e TMPDIR=/var/lib/ps-platform/judge-work \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v /var/lib/ps-platform/judge-work:/var/lib/ps-platform/judge-work \
    ps-judge-worker
```

## 4. PostgreSQL을 Backend에서만 접근 가능하도록 변경

기존에 포트가 열려 있던 PostgreSQL 컨테이너를 중지하고 제거한 후 다시 실행한다.

```bash
sudo docker run -d \
    --name ps-platform-postgres \
    --restart unless-stopped \
    --network infra_default \
    -e POSTGRES_DB=ps_platform \
    -e POSTGRES_USER=ps_user \
    -e POSTGRES_PASSWORD_FILE=/run/secrets/postgres_password \
    -v /etc/ps-platform/postgres-password:/run/secrets/postgres_password:ro,Z \
    -v infra_ps-postgres-data:/var/lib/postgresql/data \
    --health-cmd="pg_isready -U ps_user -d ps_platform" \
    --health-interval=10s \
    --health-timeout=5s \
    --health-retries=5 \
    postgres:16 \
    postgres -c timezone=UTC
```

위 명령어에는 `-p` 옵션이 없다.

> 이때 비밀번호는 파일에 저장해 경로를 지정한다.
> 
> https://hub.docker.com/_/postgres#docker-secrets

## 5. Compose

Docker 명령어를 매번 입력하는 것은 번거롭다. `compose.yaml`을 이용하면 옵션을 문서화하고 실행도 간편하게 할 수 있다.

[Composerize](https://www.composerize.com/)에 위 `docker run` 명령어들을 붙여 넣으면 `compose.yaml` 파일을 얻을 수 있다.

그 내용을 살짝 수정하여 `compose.prod.yaml` 파일로 저장한다.

```yaml
name: ps-platform

x-logging: &default-logging
    driver: json-file
    options:
        max-size: "10m"
        max-file: "3"
          
services:
    ps-backend:
        image: ps-backend:latest
        restart: unless-stopped
        depends_on:
            ps-postgres:
                condition: service_healthy
        networks:
            - app
            - database
        env_file:
            - /etc/ps-platform/backend.env
        logging: *default-logging
        
    ps-web:
        image: ps-web:latest
        restart: unless-stopped
        depends_on:
            - ps-backend
        networks:
            - app
        ports:
            - 80:80
            - 443:443
        volumes:
            - /etc/letsencrypt:/etc/letsencrypt:ro
        logging: *default-logging
        
    ps-judge-worker:
        image: ps-judge-worker:latest
        restart: unless-stopped
        depends_on:
            - ps-backend
        networks:
            - app
        env_file:
            - /etc/ps-platform/judge-worker.env
        environment:
            - TMPDIR=/var/lib/ps-platform/judge-work
        volumes:
            - /var/run/docker.sock:/var/run/docker.sock
            - /var/lib/ps-platform/judge-work:/var/lib/ps-platform/judge-work
        logging: *default-logging
        
    ps-postgres:
        restart: unless-stopped
        networks:
            - database
        environment:
            - POSTGRES_DB=ps_platform
            - POSTGRES_USER=ps_user
            - POSTGRES_PASSWORD_FILE=/run/secrets/postgres_password
        volumes:
            - /etc/ps-platform/postgres-password:/run/secrets/postgres_password:ro,Z
            - infra_ps-postgres-data:/var/lib/postgresql/data
        healthcheck:
            test: ["CMD-SHELL", "pg_isready -U ps_user -d ps_platform"]
            interval: 10s
            timeout: 5s
            retries: 5
        image: postgres:16
        command: ["postgres", "-c", "timezone=UTC"]
        logging: *default-logging

networks:
    app:
        driver: bridge
    database:
        driver: bridge
        internal: true

volumes:
    infra_ps-postgres-data:
        external: true
        name: infra_ps-postgres-data
```

문법 검사

```bash
sudo docker compose -f infra/compose.prod.yaml config --quiet
```

모든 컨테이너를 중지한 후 Compose 명령어로 다시 실행하자.

```bash
docker stop ps-web
docker stop ps-judge-worker
docker stop ps-backend
docker stop ps-platform-postgres
  
sudo docker compose -f infra/compose.prod.yaml up -d
```

## 6. 재배포 방식

```bash
sudo docker compose -f infra/compose.prod.yaml up -d --build
```

`--build` 옵션을 사용하면 이미지를 새로 빌드하고, 변경된 서비스의 컨테이너를 다시 생성한다. 별도로 `docker compose down`을 실행할 필요는 없다.

단, `--build`는 Compose 파일에 `build` 설정이 있을 때만 의미가 있다.

```yaml
services:
    backend:
      build:
        context: ../backend
      image: ps-backend:latest
```

`backend` 폴더에 `Dockerfile`이 있어야 하며, 경로는 Compose 파일을 기준으로 작성한다.

이제 `docker compose up` 명령어로 재배포할 수 있다.

## REFERENCE

- https://hub.docker.com/_/postgres#docker-secrets
- https://www.composerize.com/
