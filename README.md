## NGINX and Postgres & PG Admin and TeamCity with docker compose

# Установка docker and docker-compose

```
sudo apt update
sudo apt install curl software-properties-common ca-certificates apt-transport-https -y
curl -f -s -S -L https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu jammy stable"
sudo apt update
apt-cache policy docker-ce
sudo apt install docker-ce -y
sudo systemctl status docker
sudo apt-get install docker-compose
```


# На стороне клинета 

ssh-copy-id root@<remote_host>

# На стороне сервера

sudo nano /etc/ssh/sshd_config

```
+- PasswordAuthentication no --> yes

++ PubkeyAuthentication yes
++ ChallengeResponseAuthentication no

```
sudo systemctl reload ssh


### Installation on server
```sh
$ git clone https://github.com/epifanovmd/docker-nginx-pg-tcity.git
```

### Run
```sh
$ cd docker-nginx-pg-tcity
$ docker compose -f compose.yml -f compose-postgres.yml -f compose-teamcity.yml up --force-recreate -d
```

### Instructions for automatic deployment of the application

- The application must have a Dockerfile with instructions to run the application
- The application must have a Dockerfile with instructions to run the application
  - Project build
  - Upload to server using ssh upload
  - Creating an image based on the dockerfile and running a container based on the created image

## Example TeamCity build steps:

### First build step: (Runner type: Node.js)
```sh
yarn
npm run build:prod
```

### Second build step: (Runner type: SSH Upload)

#### Target:
```sh
94.250.254.87:/root/deployApps/lending
```

#### Authentication method: (pre-save in the project settings ssh private key)
```sh
Uploaded key
```

#### Username: (your username)
```sh
root
```

#### Select key:
```sh
id_rsa
```

#### Paths to sources:
```sh
Dockerfile
build => build
```

### Three build step: (Runner type: SSH Exec)

#### Authentication method: (pre-save in the project settings ssh private key)
```sh
Uploaded key
```

#### Username: (your username)
```sh
root
```

#### Select key:
```sh
id_rsa
```

#### Commands:
```sh
cd deployApps/lending
docker build -t lending:latest .
[[ $(docker ps -f name=lending_container -q -a) != '' ]] && docker rm --force $(docker ps -f name=lending_container -q -a)
docker run -u root -d --restart=always --network server-net -p 8082:80 --name lending_container lending:latest
docker system prune --force
```

## Example Dockerfile for run react application

```sh
FROM nginx:1.16.0-alpine
WORKDIR .
COPY build /usr/share/nginx/html

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]

```

---

License
----

MIT

**Free Software, Good Work!**
