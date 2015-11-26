nginx-lua
=========

![Docker stars](https://img.shields.io/docker/stars/ermlab/nginx-lua.png "Docker stars")
&nbsp;
![Docker pulls](https://img.shields.io/docker/pulls/danday74/nginx-lua.png "Docker pulls")

[![Docker repo](https://github.com/danday74/docker-nginx-lua/blob/master/images/docker.png?raw=true "Docker repo")](https://registry.hub.docker.com/u/danday74/nginx-lua)
&nbsp;
[![Github repo](https://github.com/danday74/docker-nginx-lua/blob/master/images/github.png?raw=true "Github repo")](https://github.com/danday74/docker-nginx-lua)

Dockerised Nginx, with Lua module, built from source + dynamic upstream like hipache (https://github.com/hipache/hipache)

Merge two repositories togather:
* https://github.com/danday74/docker-nginx-lua
* https://github.com/samalba/hipache-nginx


The docker image is based on the manual compilation instructions at ...

[http://wiki.nginx.org/HttpLuaModule#Installation](http://wiki.nginx.org/HttpLuaModule#Installation)



Useful for those who want Nginx with Lua but don't want to use OpenResty

Usage
-----

1. Create your own **Dockerfile** ...

    ```
    FROM ermlab/nginx-lua
    COPY /your/nginx.conf /nginx/conf/nginx.conf
    ```

2. Add this location block to your **nginx.conf** file

    ```
    location /hellolua {
        content_by_lua '
            ngx.header["Content-Type"] = "text/plain";
            ngx.say("hello world");
        ';
    }
    ```

    If you don't have an **nginx.conf** file then use [the conf file](https://raw.githubusercontent.com/danday74/docker-nginx-lua/master/nginx.conf) provided in the github repo

    The conf file provided is the default generated conf file with the above location block added

3. Build your docker image

4. Run your docker container - Remember to use **-p YOUR_PORT:80** in your docker run statement

5. Visit http://localhost:YOUR_PORT/hellolua



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
