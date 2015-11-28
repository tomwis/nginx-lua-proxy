nginx-lua-proxy
=========

![Docker stars](https://img.shields.io/docker/stars/ermlab/nginx-lua.png "Docker stars")
&nbsp;
![Docker pulls](https://img.shields.io/docker/pulls/ermlab/nginx-lua.png "Docker pulls")

[![Docker repo](https://github.com/Ermlab/nginx-lua-proxy/blob/master/images/docker.png?raw=true "Docker repo")](https://registry.hub.docker.com/u/danday74/nginx-lua)
&nbsp;
[![Github repo](https://github.com/Ermlab/nginx-lua-proxy/blob/master/images/github.png?raw=true "Github repo")](https://github.com/Ermlab/nginx-lua-proxy)

Dockerized Nginx+Lua dynamic proxy.

The main goal is to build the counterpart of hipache (https://github.com/hipache/hipache). The proxy try to find the host in redis database
and use it as upstream server.
The data stored in redis is in the same format as in hipache. All code is built from source.

This procject is based on wonderfull projects:
* https://github.com/danday74/docker-nginx-lua
* https://github.com/samalba/hipache-nginx

Usage
-----

1. Run REDIS database, it is essential to name it **redis**, because the lua resty redis connection objects relies on hostname=redis,

    ```
    docker run -d --name redis redis
    ```

2. Run nginx-lua-proxy container and linked it with redis

    ```
    docker run -d --link redis:redis -p 9090:80 --name $CONTAINER_NAME $IMAGE
    ```


3. Add to redis some hosts
   ```
   $ redis-cli rpush frontend:dynamic1.example.com mywebsite
   $ redis-cli rpush frontend:dynamic1.example.com http://192.168.0.50:80

   $ redis-cli rpush frontend:dynamic2.example.com mywebsite
   $ redis-cli rpush frontend:dynamic2.example.com http://192.168.0.100:80
   ```

4. Check if everything is working

   ```
   curl -H 'Host: dynamic1.example.com' http://localhost:9090

   or

   curl -H 'Host: dynamic2.example.com' http://localhost:9090
   ```

5. If you want to test in the browser you should set the dns wildcard for domain \*.example.com and it should point to your nginx proxy




VHOST Configuration
-------------------

All VHOST configuration is managed through a REDIS. This makes it possible to update the configuration
dynamically and gracefully while the server is running, and have that state
shared across workers.

Let's take an example to proxify requests to 2 backends for the hostname
`example.com`. The 2 backends IP are `192.168.0.42` and `192.168.0.43` and
they serve the HTTP traffic on the port `80`.

`redis-cli` is the standard client tool to talk to Redis from the terminal.

Follow these steps:

1. __Create__ the frontend and associate an identifier:

        $ redis-cli rpush frontend:example.com mywebsite
        (integer) 1

The frontend identifer is `mywebsite`, it could be anything.

2. __Associate__ the 2 backends:

        $ redis-cli rpush frontend:example.com http://192.168.0.42:80
        (integer) 2
        $ redis-cli rpush frontend:example.com http://192.168.0.43:80
        (integer) 3

3. __Review__ the configuration:

        $ redis-cli lrange frontend:example.com 0 -1
        1) "mywebsite"
        2) "http://192.168.0.42:80"
        3) "http://192.168.0.43:80"

While the server is running, any of these steps can be re-run without messing up
with the traffic.

Automated
---------

The master branch on the github repo is watched by an automated docker build

Which builds docker image **ermlab/nginx-lua** on push to master

On success, the docker build triggers the docker repo's webhooks (if any)

License
-------

[http://www.apache.org/licenses/LICENSE-2.0](http://www.apache.org/licenses/LICENSE-2.0)

APACHE LICENSE-2.0 ... In other words, please use freely and do whatever you want with it!
