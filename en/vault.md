Here at [Codelitt](http://www.codelitt.com/) we are really concerned about security. We've writen about it before [here](http://www.codelitt.com/blog/pragmatic-approach-building-ruby-rails-apps-quickly-quality-code/#security), but there are still a lot of practices and services that we use in our ecosystem. Today I'm going to talk about [Vault](vaultproject.io), the service that we use to handle sensible env variables.

[Vault](vaultproject.io) is an awesome service to store sensitive key/values. Here we use it to store our env variables like service keys, service password and so on. 

Let's learn how to set it up.

In case you are setting up a new server take a look in our [server security practices](https://github.com/codelittinc/incubator-resources/blob/master/best_practices/servers.md) and remember to *not* set it up with the root admin.

If you don't have the docker-compose installed take a look on this [great tutorial](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-ubuntu-14-04) provided by [Digital Ocean](https://www.digitalocean.com/).

We are going to set it up with the `deploy` user

1 - Before start with the containers let's setup the configuration. Put the following content in `/home/deploy/vault/vault.config`

```
listener "tcp" {
  address = "0.0.0.0:9000"
  tls_disable = 1
}

backend "consul" {
  address = "172.18.0.2:8500"
  path = "vault"
}
```

2 - As we are using docker the initial setup is very simple, we are going to use a Vault image, you can find the source [here](https://github.com/cgswong/docker-vault). Put the following content in your `docker-compose.yml`

```
version: '2'
services:
  # Vault
  consul:
    container_name: consul
    image: progrium/consul
    restart: always
    hostname: consul
    ports:
      - 8500:8500
    command: "-server -bootstrap"

  vault:
    container_name: vault
    image: cgswong/vault
    restart: always
    volumes:
      - '/home/deploy/vault/vault.config:/root/vault.config'
    ports:
      - 8200:9000
    environment:
      VAULT_ADDR: 'http://0.0.0.0:9000'
    cap_add:
      - IPC_LOCK
    depends_on:
      - consul
    command: "server -config /root/vault.config"
```

We are going to use Consul as our secret back-end, you can find more info about it in the [Vault Docs](https://www.vaultproject.io/docs/secrets/consul/index.html).

3 - Let's run the containers now. run: `docker-compose up -d consul`

This command will only run the consul container, before starting the Vault one you need to update the consul ip in the Vault config.

Run `docker inspect consul` and you will see something like: `'IPAddress': '172.18.0.4'` take the ip and set it up and replace the address ip on the config: 

```
backend "consul" {
  address = "172.18.0.2:8500"
  path = "vault"
}
```

Now run `docker-compose up`

Congratulations! now you have Vault working!

To enable it you need to take some steps, they are very well documented in the Initializing section in the [docs](https://www.vaultproject.io/intro/getting-started/deploy.html)

Now you are able to store your keys, but calm down! Let's improve the security of your server.

First take a look in how we work with nginx [here](add link) and update your docker-compose to:

```
version: '2'
services:
  # Vault
  consul:
    container_name: consul
    image: progrium/consul
    restart: always
    hostname: consul
    ports:
      - 8500:8500
    command: "-server -bootstrap"

  vault:
    container_name: vault
    image: cgswong/vault
    restart: always
    volumes:
      - '/home/deploy/vault/vault.config:/root/vault.config'
    ports:
      - 8200:9000
    environment:
      VAULT_ADDR: 'http://0.0.0.0:9000'
    cap_add:
      - IPC_LOCK
    depends_on:
      - consul
    command: "server -config /root/vault.config"
    
  nginx:
    image: nginx
    container_name: nginx
    restart: always

    ports:
     - '80:80'
     - '443:443'
     - '9000:9000'
 
    links:
      - graylog

    volumes: 
     - /etc/nginx-docker/:/etc/nginx/   
```

As Vault is a application that needs a really high security you **SHOULD** use HTTPS.

4 - A security good practice for this kind of application is to limit the access to the Vault port to be open to the application that will store/fetch the keys only. In order to have it you can either user your server application firewall (like aws) or you can run:

`iptables -I PREROUTING 1 -t mangle ! -s your_application_ip -p tcp --dport 12201 -j DROP`

Ba-Du-Tss!

Now you have an application where you can store your sensitive key/values!

If you want to know more about our best practices, take a look in our [repository](https://github.com/codelittinc/incubator-resources). We open source all of our policies and best practices as well as continue to add to them there.
