Here at [Codelitt Incubator](http://www.codelitt.com/) we really prefer to self-host our services when possible, especially for sensitive projects. We have used LogEntries for quite a few projects, but needed a self-hosted option for a few products. We did a ton of research and finally settled on [Graylog](https://www.graylog.org/). So far it has been the best self-hosted tool to centralize logs that we've found. It has a rich web client with a lot of filters and a solid UI. They offer good documentation to set it up, however, because we strive to have [12Factor](12factor.net) compliant applications we've organized a small HOW-TO to setup Graylog with Docker + Docker-Compose.

If you are setting up a new server take a look at our [server security practices](https://github.com/codelittinc/incubator-resources/blob/master/best_practices/servers.md). This should provide good foundation. **Remember to *not* set it up with the root user.**

If you don't have docker-compose installed take a look here [great tutorial](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-ubuntu-14-04) provided by [Digital Ocean](https://www.digitalocean.com/). They give an overview of Docker Compose, setting it up, and how to use it to orchestrate microservices. You really only need the first 2 steps, but it's a very short read and I recommend taking a look at the others to get a feel for it. 

After you've got that setup, let's jump into the [Graylog](https://www.graylog.org/) setup:

###1 - Setup the graylog config folder on your server
```
mkdir -p graylog/config
cd graylog/config
wget https://raw.githubusercontent.com/Graylog2/graylog2-images/2.0/docker/config/graylog.conf
wget https://raw.githubusercontent.com/Graylog2/graylog2-images/2.0/docker/config/log4j2.xml
```

To grant read/write permissions run: `sudo chmod -R a+rw graylog/`

###2 - Setup your Docker Compose file

In their [docs](http://docs.graylog.org/en/2.0/pages/installation/docker.html) they provide a docker compose file which we are going to change a bit.

You'll want to use the following setup in the `docker-compose.yml` file. Take note to replace passwords:
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
A bit of explanation about what we just added to the compose file. 

Graylog needs at least 3 ports to work properly:

`9000` is the front client, to access the web version you will access http://yourserver.com:9000

`12900` is the server port which will provide the data and authentication for the web client.

`12201/udp` is the port of one of our log inputs. As you can see we oppening a UDP port, it happens because we are going to send the logs to the application using the docker-compose Gaylog integration, which uses this protocol.

If you are going to use more than one log source remember to add it as `portnumber:udp`

###3 - Start docker compose

To spin it up run: `docker-compose up -d`

If you run `docker ps -a` you will see that the graylog container is not running, it happens because of the `GRAYLOG_REST_TRANSPORT_URI` env variable. The value in the docker-compose file needs to be updated with the current container ip.  Run:

`docker inspect graylog` 

You will see something like: `'IPAddress': '172.18.0.4'` take the ip and replace the `127.0.0.1` from the `     GRAYLOG_REST_TRANSPORT_URI: http://127.0.0.1:12900` with it.

Run `docker-compose up -d` again

###4 - Setup nginx

First take a look in how we use nginx [here](add link) with Docker Compose and update your `docker-compose.yml` to:
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
      
  nginx:
    image: nginx
    container_name: nginx
    restart: always

    ports:
     - '80:80'
     - '443:443'
     - '9000:9000'
     - '12900:12900'
     - '12201/udp:12201/udp'

    links:
      - graylog

    volumes: 
     - /etc/nginx-docker/:/etc/nginx/
```

Take the content of the nginx `default.conf` [here](https://github.com/kaiomagalhaes/blog/blob/master/en/graylog/default.conf) and update the `docker_container_ip` with your graylog container ip.

Now you should be able to access it with: https://yourserver.com:9000

The default user is
```
user: admin
password: admin
```

In order to change the admin password you need to update the env variable `GRAYLOG_ROOT_PASSWORD_SHA2` with the content of: `echo -n yourpassword | shasum -a 256`

###5 - Set up the log input

Right now you have Graylog running but it is useless without the logs from your application. To add them you have two options:

 1. Use Graylog docker integration
 2. Use a library like [Ruby GELF](https://github.com/Graylog2/gelf-rb)
 
The [12Factor](http://12factor.net/logs) says:

"A twelve-factor app never concerns itself with routing or storage of its output stream."

To follow that philosophy, we should use the first option, which is very easy with docker-compose. You just need to add the following snippet inside the service that you want to log in your docker-compose.yml file.

```
    logging:
      driver: "gelf"
      options:
        gelf-address: "udp://ip_of_your_graylog_application:12201"
        gelf-tag: "my application"
```

###6 - Securing your graylog application

As Graylog doesn't offer an application_key you need to update your iptables to allow only inputs from your application server. To do this, you need to run:

`iptables -I PREROUTING 1 -t mangle ! -s your_application_ip -p tcp --dport 12201 -j DROP`


Ba-du-tss!

You have a secure server application running!

If you want to know more about our best practices, take a look in our [repository](https://github.com/codelittinc/incubator-resources). We open source all of our policies and best practices as well as continue to add to them there.
