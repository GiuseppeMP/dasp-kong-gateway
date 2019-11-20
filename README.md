# dasp-kong-gateway

Desenvolvimento Ágil Starter Pack - Kong Gateway

## Install

1.Create a Docker network

This is the same as in the Pg/Cassandra guide. We’re also using kong-net as the network name and it can also be changed to something else.

`$ docker network create kong-net`

This step is not strictly needed for running Kong in DB-less mode, but it is a good precaution in case you want to add other things in the future (like a rate-limiting plugin backed up by a Redis cluster).

2.Create a Docker volume

For the purposes of this guide, a Docker Volume is a folder inside the host machine which can be mapped into a folder in the container. Volumes have a name. In this case we’re going to name ours kong-vol

`$ docker volume create kong-vol`

You should be able to inspect the volume now:

`$ docker volume inspect kong-vol`

The result should be similar to this:

`[ { "CreatedAt": "2019-05-28T12:40:09Z", "Driver": "local", "Labels": {}, "Mountpoint": "/var/lib/docker/volumes/kong-vol/_data", "Name": "kong-vol", "Options": {}, "Scope": "local" } ]`

Notice the MountPoint entry. We will need that path in the next step.

3.Prepare your declarative configuration file

The syntax and properties are described on the Declarative Configuration Format guide.

Add whatever core entities (Services, Routes, Plugins, Consumers, etc) you need there.

On this guide we’ll assume you named it kong.yml.

Save it inside the MountPoint path mentioned in the previous step. In the case of this guide, that would be /var/lib/docker/volumes/kong-vol/data/kong.yml

4.Start Kong in DB-less mode

Although it’s possible to start the Kong container with just KONG_DATABASE=off, it is usually desirable to also include the declarative configuration file as a parameter via the KONG_DECLARATIVE_CONFIG variable name. In order to do this, we need to make the file “visible” from within the container. We achieve this with the -v flag, which maps the kong-vol volume to the /usr/local/kong/declarative folder in the container.

\$ docker run -d --name kong \
 --network=kong-net \
 -v "kong-vol:/usr/local/kong/declarative" \
 -e "KONG_DATABASE=off" \
 -e "KONG_DECLARATIVE_CONFIG=/usr/local/kong/declarative/kong.yml" \
 -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
 -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
 -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
 -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
 -e "KONG_ADMIN_LISTEN=0.0.0.0:8001, 0.0.0.0:8444 ssl" \
 -p 8000:8000 \
 -p 8443:8443 \
 -p 8001:8001 \
 -p 8444:8444 \
 kong:latest

5.Use Kong

Kong should be running and it should contain some of the entities added in kong.yml:

`$ curl -i http://localhost:8001/`

For example, get a list of services:

`$ curl -i http://localhost:8001/services`
