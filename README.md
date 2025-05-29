# DOKUMENTACJA SERWER CI/CD

## 1. Cel
Opis serwera CI/CD i współpracy usług Jenkins, GitLab oraz Nginx w środowisku Docker Compose

## 2. Środowisko
+ Jenkins - serwer CI/CD opowiedzialny za budowę projektu
+ Gitlab - repozytorium kodu z webowym interfejsem do zarządzania wersjami i kodem źródłowym
+ Nginx - reverse proxy obsługujące kierowanie ruchem do Jenkins i GitLab
***
+ Jenkins - http://jenkins.localhost/
+ Gitlab - http://gitlab.localhost/

## 3. Konfiguracja Docker Compose
```
version: '3.8'

services:
  gitlab:
    image: gitlab/gitlab-ce:latest
    container_name: gitlab
    restart: always
    hostname: 'gitlab.local'
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://gitlab.localhost/'
    ports:
      - '8080:80'   
      - '2222:22'    
    volumes:
      - ./gitlab/config:/etc/gitlab
      - ./gitlab/logs:/var/log/gitlab
      - ./gitlab/data:/var/opt/gitlab
    networks:
      - ci_net

  jenkins:
    build:
      context: .
      dockerfile: Dockerfile.jenkins
    container_name: jenkins
    restart: always
    user: root
    hostname: 'jenkins.local'
    ports:
      - '8081:8080' 
      - '50000:50000'
    volumes:
      - ./jenkins_home:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock
    networks:
      - ci_net

  nginx:
    image: nginx:latest
    container_name: nginx_proxy
    restart: always
    ports:
      - "80:80"         
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro  
    depends_on:
      - gitlab
      - jenkins
    networks:
      - ci_net 

networks:
  ci_net:
    driver: bridge
```

### Dockerfile.jenkins
```
FROM jenkins/jenkins:lts

USER root

RUN apt-get update && \
    apt-get install -y \
        cmake \
        clang \
        build-essential \
        libsdl2-dev \
        libglew-dev \
        libgl1-mesa-dev \
        pkg-config \
        glew-utils && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

USER jenkins
```

Ze względu na to, że kod, który służył mi jako test do budowania zawierał kilka bibliotek zewnętrzych musiałem je zainstalować wewnąrz kontenera jenkins, aby mógł wszystko skompilować.

## Konfiguracja nginx
```
events {}

http {
    server {
        listen 80;

        server_name gitlab.localhost;

        location / {
            proxy_pass http://gitlab:80/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;

            proxy_redirect off;
        }
    }

    server {
        listen 80;

        server_name jenkins.localhost;

        location / {
            proxy_pass http://jenkins:8080/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;

            proxy_redirect off;
        }
    }
}
```
## Pipeline jenkins
```
pipeline {
    agent any

    stages {
        stage('Prepare') {
            steps {
                sh 'mkdir -p build'
                echo "Katalog build utworzony"
            }
        }


        stage('CMake') {
            steps {
                dir('build') {
                    sh 'cmake ..'
                    echo "Budowanie cmake"
                }
            }
        }

        stage('Build') {
            steps {
                dir('build') {
                    sh 'make'
                    echo "Budowanie make"
                }
            }
        }
    }

    post {
        always {
            echo 'Projekt zbudowany.'
        }
    }
}
```

## Uruchamianie
`docker-compose build jenkins` `docker-compose up -d`

**Pełne włącznie chwilę trwa ze względu na gitlaba, który potrzebuje trochę czasu przy uruchomieniu**

## Specyfikacja
### Ubuntu 24.04
### Docker 26.1.3
### Docker Compose 1.29.2
### Jenkins 2.504.1
+ Login: admin
+ Hasło: admin
### Nginx 1.27.5
### GitLab 18.0
+ Login: root
+ Hasło: root
### MySQL 8.0.42
+ Login: root
+ Hasło: root

# Problemy
Problemów dużo nie było, a jak już występowały to ze względu na nginx. Chociaż w trakcie zmieniłem Gitea na GitLaba.
