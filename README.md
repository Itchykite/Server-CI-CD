# Server-CI-CD

Serwer jenkins + gitea
reverse proxy nginx localhost/jenkins, localhost/git

## Must have
1. Docker
2. Docker-compose
3. MySQL

## Instalation
1. `git clone https://github.com/Itchykite/Server-CI-CD.git`
2. `sudo docker-compose up -d`

### Gitea config
Go on http://localhost/git
Config gitea as you want
If problems check app.ini

`sudo docker exec -it gitea sh`
`vi /data/gitea/conf/app.ini`

### Jenkins config
This is accualy easy...
