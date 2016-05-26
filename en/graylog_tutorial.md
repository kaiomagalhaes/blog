Here at [Codelitt](http://www.codelitt.com/) we use the best self-hosted tool to centralize logs that we've found, which is  [Graylog](graylog.org). It offers a rich web client with a lot of filters and graphics.

To set it up they offer a good documentation, but as we strive to have [12Factor](12factor.net) compliant services and applications we've organized a HOW-TO with it and docker&docker-compose.

In case you are setting up a new server take a look in our [server security practices](https://github.com/codelittinc/incubator-resources/blob/master/best_practices/servers.md). 

Remember to not set it up with the root admin.

If you don't have the docker-compose installed take a look on this [great tutorial](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-ubuntu-14-04) provided by [Digital Ocean](https://www.digitalocean.com/).

And let's go to the Graylogs setup:

1 - Setup the graylogs config folder
```
mkdir -p graylog/config
cd graylog/config
wget https://raw.githubusercontent.com/Graylog2/graylog2-images/2.0/docker/config/graylog.conf
wget https://raw.githubusercontent.com/Graylog2/graylog2-images/2.0/docker/config/log4j2.xml
```

to grant read/write permissions run: `sudo chmod -R a+rw graylog/`

2 - In their [docs](http://docs.graylog.org/en/2.0/pages/installation/docker.html) they provide a docker compose file which we are going to change a bit.

Put the following content in the `docker-compose.yml` file.
```
version: '2.0'
services:
  mongo:
    image: "mongo:3"
    container_name: mongo
    volumes:
      - /home/deploy/data/mongo:/data/db

  elastic:
    image: "elasticsearch:2"
    container_name: elastic
    command: "elasticsearch -Des.cluster.name='graylog'"
    volumes:
      - /home/deploy/data/elasticsearch:/usr/share/elasticsearch/data

  graylog:
    image: graylog2/server:2.0.0-1
    container_name: graylog
    volumes:
      - /home/deploy/data/journal:/usr/share//home/deploy/data/journal
      - /home/deploy/config:/usr/share//home/deploy/data/config

    environment:
      GRAYLOG_PASSWORD_SECRET: somepasswordpepper
      GRAYLOG_ROOT_PASSWORD_SHA2: 8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918
      GRAYLOG_REST_TRANSPORT_URI: http://127.0.0.1:12900

    links:
      - mongo:mongo
      - elastic:elasticsearch
    ports:
      - "9000:9000"
      - "12900:12900"
      - "12201/udp:12201/udp"
```

Graylog needs at least 3 ports to work properly:

`9000` is the front client, so to access the web version you will access http://youraserver.com:9000

`12900` is the server port which will provide the data and authentication for the web client.

`12201/udp` is the port of one of our log inputs. As you can see we oppening a UDP port, it happens because we are going to send the logs to the application using the docker-compose Gaylog integration, which uses this protocol.

If you are going to use more than one log source remember to add it as `portnumber:udp`

3 - To spin it up run: `docker-compose up -d`

if you run `docker ps -a` you will see that the graylog container is not running, it happens because of the `GRAYLOG_REST_TRANSPORT_URI` env variable. The value in the docker-compose file needs to be updated with the current container ip.
run: `docker inspect graylog`. You will see something like: `'IPAddress': '172.18.0.4'` take the ip and set it up and replace the `127.0.0.1` from the `     GRAYLOG_REST_TRANSPORT_URI: http://127.0.0.1:12900`.

run `docker-compose up -d` again

4 - Let's setup nginx, update your docker-compose to:
```

```