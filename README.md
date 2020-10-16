# Note

The dockerfiles images and procedure are still viables, but wont by more updates on this repo. We migrate the project to the official PandoraFMS repository 

Github: https://github.com/pandorafms/pandorafms/tree/develop/extras/docker/centos8


Dockerhub: https://hub.docker.com/r/pandorafms/pandorafms-open-stack-el8


# Pandora FMS

```
docker run --name Pandora_new %container_name% --rm \
-p %local_httpd_port%:80 \
-p %local_tentacle_port%:41121 \
-e DBHOST=%Mysql_Server_IP% \
-e DBNAME=%database_name% \
-e DBUSER=%Mysql_user% \
-e DBPASS=%Mysql_pass% \
-e DBPORT=%Mysql_port% \
-e INSTANCE_NAME=%server name% \
-ti rameijeiras/pandorafms-community
```
Example:
```
docker run --name Pandora_new --rm \
-p 8081:80 \
-p 41125:41121 \
-e DBHOST=192.168.80.45 \
-e DBNAME=pandora_demos_1 \
-e DBUSER=pandora \
-e DBPASS=pandora \
-e DBPORT=3306 \
-e INSTANCE_NAME=pandora201 \
-ti rameijeiras/pandorafms-community
```

### Integrated database for PandoraFMS container
There is a preconfigured database image in this repo to connect the Pandora environment  so you can up the database and then point the pandora container to the database.

Example:
```
docker run --name Pandora_DB \
-p 3306:3306 \
-e MYSQL_ROOT_PASSWORD=pandora \
-e MYSQL_DATABASE=pandora \
-e MYSQL_USER=pandora \
-e MYSQL_PASSWORD=pandora \
-d rameijeiras/pandorafms-percona-base
```

This creates a Percona mysql docker and a database called Pandora with grants to the pandora user (optional) and the credentials for root user. 
In this example we expose the 3308 for database connection. 

Using this configuration (getting the container ip from DB container ip) you can execute the next container pandora pointing to it:

```
docker run --name Pandora_new --rm \
-p 8081:80 \
-p 41125:41121 \
-e DBHOST=<percona container ip> \
-e DBNAME=pandora \
-e DBUSER=pandora \
-e DBPASS=pandora \
-e DBPORT=3306 \
-e INSTANCE_NAME=pandora_inst \
-ti rameijeiras/pandorafms-community
```

### Docker Compose Stack

if you want to run an easy to deploy stack you may use the docker-compose.yml file

```
version: '3.1'
services:
  db:
    image: rameijeiras/pandorafms-percona-base
    restart: always
    command: ["mysqld", "--innodb-buffer-pool-size=300M"] 
    environment:
      MYSQL_ROOT_PASSWORD: pandora
      MYSQL_DATABASE: pandora
      MYSQL_USER: pandora
      MYSQL_PASSWORD: pandora
    networks:
     - pandora

  pandora:
    image: rameijeiras/pandorafms-community
    restart: always
    depends_on:
      - db
    environment:
      MYSQL_ROOT_PASSWORD: pandora
      DBHOST: db
      DBNAME: pandora
      DBUSER: pandora
      DBPASS: pandora
      DBPORT: 3306
      INSTANCE_NAME: pandora01
      SLEEP: 5
      RETRIES: 3
    networks:
     - pandora
    ports:
      - "8080:80"
      - "41121:41121"
      - "162:162/udp"

networks:
  pandora:
```
just by running: `docker-compose -f PandoraFMS/pandorafms_community/docker-compose.yml up`
