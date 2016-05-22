Here at [Codelitt](codelitt.com) we use Nginx as our proxy server. Commonly we install nginx in the server and run the applications with docker and docker-compose. As we strive to have a configuration that doesn't depends on the server now we are using it with docker and docker-compose.

To set it up up you can use the following docker compose:

```
version: '2.0'
services:
  nginx:
    image: nginx
    container_name: nginx
    restart: always

    ports:
     - '80:80'
     - '443:443'

    links:
      - app

    volumes: 
     - /etc/nginx-docker/:/etc/nginx/
```

As you can see the configuration files will be in a host folder, here we use `/etc/nginx-docker`. Inside this folder you need to add the files that you can find [here](https://github.com/kaiomagalhaes/nginx-docker-configuration)

To have a SSL
