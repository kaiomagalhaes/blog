Since the [Beginning of the time](https://upload.wikimedia.org/wikipedia/commons/c/c2/Lambda-Cold_Dark_Matter,_Accelerated_Expansion_of_the_Universe,_Big_Bang-Inflation.jpg) we have been sufering the **'But it is working in my machine'** curse. It happens because we are probably not going to deploy the application in our programmer computer and it happens to be 
different than the production one, how can it be so different? In a lot of ways e.g: operational system, environment variables, installed dependencies and the list goes far beyond it.

To minimize this problem here at [Codelitt](http://codelitt.com) we develop and deploy our applications inside docker containers.

What is Docker you ask? Let's use the description that they provide in their [site](https://www.docker.com/what-docker):

```
Docker containers wrap up a piece of software in a complete filesystem that 
contains everything it needs to run: code, runtime, system tools, 
system libraries – anything you can install on a server. This guarantees that it 
will always run the same, regardless of the environment it is running in.
```
**"It will always run the same"** it is like a salve in our old wounds.

So using it in the development and production environments we can assure that they will be the same and that ghost of "It works on my machine" will fade alway.

Here we are going to explain how we develop and setup our servers to production.

In this tutorial we are going use a Ruby on Rails application as example.

First you need to [install docker](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-16-04)

Development

All of our applications have at least one Docker container, which has all the necessary information to build it.

#### Development

```
# This is our DEVELOPMENT dockerfile.
#
# This uses Codelitt's Ruby 2.2 image found at:
# https://github.com/codelittinc/dockerfiles/blob/master/ruby/Dockerfile
FROM codelittinc/ruby:2.2
MAINTAINER Codelitt, Inc.

# Mount any shared volumes from host to container @ /share
VOLUME ["/share"]

# Install dependencies and rails-api
RUN apt-get update \
    && apt-get install -y nodejs \
    && rm -rf /var/lib/apt/lists/* \
    && gem install --no-rdoc --no-ri \
    rails-api

WORKDIR /share
```

It is a dockerfile for a ruby application. Add it inside your application and build it. [Here](https://docs.docker.com/mac/step_four/) more informations about how to do it.

It is simple as that. Now when you format your computer you only need to install docker and create your container. You no more need to install all the dependencies of your 20 projects.

#### Production

You have developed your application and now you want to set it up so you client can access it. 

Awesome now you need to worry about the security in your server, we have made a good tutorial about it [here](http://www.codelitt.com/blog/my-first-10-minutes-on-a-server-primer-for-securing-ubuntu/).

The production dockerfile for the same application would be:

```
# This is our Production dockerfile.
#
# This uses Codelitt's Ruby 2.2 image found at:
# https://github.com/codelittinc/dockerfiles/blob/master/ruby/Dockerfile
FROM codelittinc/ruby:2.2
MAINTAINER Codelitt, Inc.

# Mount any shared volumes from host to container @ /share
VOLUME ["/share"]

# Install dependencies and rails-api
RUN apt-get update \
    && apt-get install -y nodejs \
    && rm -rf /var/lib/apt/lists/* \
    && gem install --no-rdoc --no-ri rails-api \
    && gem install --no-rdoc --no-ri puma

WORKDIR /share
ADD Gemfile /share/Gemfile
ADD Gemfile.lock /share/Gemfile.lock
RUN bundle install
ADD ./ /share

CMD cp config/database.yml.example config/database.yml
ENV RAILS_ENV production
ENV SECRET_KEY_BASE MY_UNSAFE_SECRET
CMD rails s Puma -b 0.0.0.0 -e production -p 4000
```

As it is a rails application you need to change the **SECRET_KEY_BASE** for another hash (unless you want us to know your application secret key)

If you are using UFW in your server (like we recommend) you should take a look on this [docker documentation](https://docs.docker.com/engine/installation/linux/ubuntulinux/#enable-ufw-forwarding) as well.

Build the container and there is your application running on production.

#### Summary

Now that you know the basics of docker you are able to develop in a alike production environment, however it is not because you running inside a docker container that your application is safe. [Here](www.codelitt.com/blog/pragmatic-approach-building-ruby-rails-apps-quickly-quality-code/) we explain how to improve security and a lot of other things for ruby/rails applications and as soon as possible we will post how we deploy our applications with security.

If you want to know more about our best practices, take a look in our repository. We open source all of our policies and best practices as well as continue to add to them there.
