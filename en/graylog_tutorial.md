Here at [Codelitt](http://www.codelitt.com/) we decided to use a tool to centralize our logs. The best self hosted version that we found was [Graylog](graylog.org). It offers a rich web client with a lot of filters and graphics.

To set it up they offer a good documentation, but as we strive to have [12Factor](12factor.net) compliant services and applications we've organized a HOW-TO with it and docker&docker-compose.

In case you are setting up a new server take a look in our [server security practices](https://github.com/codelittinc/incubator-resources/blob/master/best_practices/servers.md). 

Remember to not set it up with the root admin.

If you don't have the docker-compose installed take a look on this [great tutorial](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-ubuntu-14-04) provided by [Digital Ocean](https://www.digitalocean.com/).

And let's go to the Graylogs setup:

1 - Setup the graylogs config folder
```
mkdir -p /graylog/config
cd /graylog/config
wget https://raw.githubusercontent.com/Graylog2/graylog2 images/2.0/docker/config/graylog.conf
wget https://raw.githubusercontent.com/Graylog2/graylog2-images/2.0/docker/config/log4j2.xml
```

2 - In their [docs](http://docs.graylog.org/en/2.0/pages/installation/docker.html) they provide a docker compose file which we are going to change a bit.

Put the following content in the docker-compose.yml file.
```
some-mongo:
  image: "mongo:3"
  volumes:
    - /graylog/data/mongo:/data/db
some-elasticsearch:
  image: "elasticsearch:2"
  command: "elasticsearch -Des.cluster.name='graylog'"
  volumes:
    - /graylog/data/elasticsearch:/usr/share/elasticsearch/data
graylog:
  image: graylog2/server:2.0.0-1
  volumes:
    - /graylog/data/journal:/usr/share/graylog/data/journal
    - /graylog/config:/usr/share/graylog/data/config
  environment:
    GRAYLOG_PASSWORD_SECRET: somepasswordpepper
    GRAYLOG_ROOT_PASSWORD_SHA2: 8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918
    GRAYLOG_REST_TRANSPORT_URI: http://127.0.0.1:12900

  links:
    - some-mongo:mongo
    - some-elasticsearch:elasticsearch
  ports:
    - "9000:9000"
    - "12900:12900"
    - "12201/udp:12201/udp"
```





