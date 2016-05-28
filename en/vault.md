Here at [Codelitt](http://www.codelitt.com/) we are really concerned about security. We've writen about it before [here](http://www.codelitt.com/blog/pragmatic-approach-building-ruby-rails-apps-quickly-quality-code/#security), but there are still a lot of practices and services that we use in our ecosystem. Today I'm going to talk about [Vault](vaultproject.io), the service that we use to handle sensible env variables.

[Vault](vaultproject.io) is an awesome service to store sensitive key/values. Here we use it to store our env variables like service keys, service password and so on. 

So let's learn how to set it up.

In case you are setting up a new server take a look in our [server security practices](https://github.com/codelittinc/incubator-resources/blob/master/best_practices/servers.md) and remember to *not* set it up with the root admin.

If you don't have the docker-compose installed take a look on this [great tutorial](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-ubuntu-14-04) provided by [Digital Ocean](https://www.digitalocean.com/).



1 - 

