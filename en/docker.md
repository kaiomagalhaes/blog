Since the [beginning of the time](https://upload.wikimedia.org/wikipedia/commons/c/c2/Lambda-Cold_Dark_Matter,_Accelerated_Expansion_of_the_Universe,_Big_Bang-Inflation.jpg) (Disclaimer: possibly not an accurate timeline), we have been suffering from the *'But it's working in my machine'* curse. It happens because we are probably not going to deploy the application in our local machine. Our local machine usually happens to be different than the production one. I can be different in lots of ways e.g: operational system, environment variables, installed dependencies and the list continues. 

To minimize this problem here at [Codelitt](http://codelitt.com) we develop and deploy our applications inside docker containers. It started as a bit of a grassroots movement among employees, but the movement has really taken hold. From simple Jekyll pages, to frontend clients, to blue-green deployment for services, we've Dockerized **everything**.

![Dockerize all the things](https://i.imgur.com/FiNj8.jpg)

For those not terribly family with Docker, you can read more on their [site](https://www.docker.com/what-docker), but from the horses mouth:

```
Docker containers wrap up a piece of software in a complete filesystem that 
contains everything it needs to run: code, runtime, system tools, 
system libraries – anything you can install on a server. This guarantees that it 
will always run the same, regardless of the environment it is running in.
```
There are many benefits of Docker, but what has always stood out to us is the fact **"it will always run the same on any machine"**. It is like a salve in our old wounds. That ghost of "It works on my machine" fades away and we can focus on things that matter.

We wanted to explain how we develop and setup our applications to production with Docker. We have a set of best practices and have automated our deployment and implemented [blue-green deploys](http://martinfowler.com/bliki/BlueGreenDeployment.html) with it, but in this first article we're going to cover getting Dockerfiles setup for a project. Dockerfiles are just a set of instructions which build the system in a predictable and repeatable way across engineers' machines (and production machines).  

In this tutorial we are going use a Ruby on Rails application as example.

First you need to [install docker](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-16-04). It's straight forward enough so we'll not cover that for now. 

All of our applications have at least one Docker container, which has all the necessary information to build it -- most have a few. Docker really becomes powerful when you start to use a container per process. It separates concerns, allows for great scalability, agility, resilience, and speed. Containers can be easily coordinated with Docker Compose, but we'll cover that in another article. For now let's setup two Dockerfiles. One for development and one for prod 

#### Development

`Dockerfile.development`
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
    && rm -rf /var/lib/apt/lists/* \
    && gem install --no-rdoc --no-ri \
    rails-api

WORKDIR /share

CMD ["/bin/bash", "-l"]
```

It is a Dockerfile for a Ruby application. Add it to you application's directory and build it. [Here](https://docs.docker.com/mac/step_four/) more information about how to do it.

### Running

As it is a Ruby on Rails application you are probably going to need a database. Now you may be asking "Do I need to install a database on my machine?" and deep in your thoughts you know the answer, you don't need.

Let's run a postgres database on docker

`docker run --name myapp_db -e POSTGRES_PASSWORD=postgres -d postgres`

This command will download a postgres image and run it, as well as open a the por 5432 for conections.

Now let's run the application.

First you need to build the application

`docker build -t myapp .`


And now you run it

```
docker run -d \
-ti \
--name myapp \
-e MYAPP_DATABASE_PASSWORD=postgres \
-e MYAPP_DATABASE_USER=postgres \
-v $(pwd):/share \
-p 3000:3000 \
--link myapp_db:db myapp /bin/bash -l
```

Now you have the machine running. Oh but how do I access it? It is easy, just run:

`docker exec -it myapp bash`

As you have access to your application you just need to setup it e.g: `bundle install`, `rake db:create` and so on.


It is simple as that. Now when you format your computer, work with a teammate, or switch from project to project, you only need to fire up a container. You don't need to install all the dependencies of your 20 projects to your host machine. All of the environment is contained in the container.

#### Production

When you're ready to move to production, you'll still want to secure your host server. Containers are only as secure as their host. We have a good tutorial about our [first 10 minutes on a server](http://www.codelitt.com/blog/my-first-10-minutes-on-a-server-primer-for-securing-ubuntu/) which should take care of most basic server security concerns.

The production Dockerfile for the same application would be:

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

As it is a Rails application you'll need to change the **SECRET_KEY_BASE** to another hash (unless you want us to know your application secret key)

If you are using UFW in your server [(like we recommend)](http://www.codelitt.com/blog/my-first-10-minutes-on-a-server-primer-for-securing-ubuntu/) you should take a look at this [Docker documentation](https://docs.docker.com/engine/installation/linux/ubuntulinux/#enable-ufw-forwarding) as well.

Build the container and there is your application running in production. 

#### Summary

Now that you know the basics of Docker you are able to develop in a near identical environment as your production environment. We'll be following up on this article with follow ups on Docker compose to coordinate microservices, Docker blue-green deployment, how we do one command deploys, and how to secure Docker containers. In the mean time, take a look at our aforementioned server security article or [our pragmatic approach to building quality and secure Rails apps](www.codelitt.com/blog/pragmatic-approach-building-ruby-rails-apps-quickly-quality-code/). We explain how to improve security, code smell, and code quality for ruby/rails applications. 

We're making a big push to externalize our processes and best practices, if you're interested in learning more take a look at our [repository](https://github.com/codelittinc/incubator-resources). We open source all of our policies and best practices as well as continue to add to them there.
