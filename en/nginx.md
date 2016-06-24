Here at [Codelitt Incubator](codelitt.com) we use Nginx as our proxy server. Commonly we install nginx on the server and run the applications with docker and docker-compose. As we strive to have a configuration that isn't server based now we are using it within a docker container similar to our applications.

To set it up up you can use the following docker compose config:

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
      - yourappservice

    volumes: 
     - /etc/nginx-docker/:/etc/nginx/
```

As you can see the configuration files will be in a host folder. We use: `/etc/nginx-docker`.
 Inside this it you need to add the nginx config files that you can find [here](https://github.com/kaiomagalhaes/nginx-docker-configuration). Don't forget to set up the `default.conf` that is localized in `/etc/nginx-docker/conf.d/default.conf`

If you want to listen to another ports make sure to add them in the `ports` section of the docker-compose file.

If you'd like to use an SSL configuration you can either [create a self signed ssh key](http://www.akadia.com/services/ssh_test_certificate.html) or use [Let's Encrypt](https://letsencrypt.org/getting-started/). 

If you want to know more about our best practices, take a look in our [repository](https://github.com/codelittinc/incubator-resources). We open source all of our policies and best practices. We have quite a few published already and are working on publishing more.
