This page contains a list of commonly used `docker compose` commands and flags.

## [](https://github.com/collabnix/dockerlabs/blob/ce94351e2ba629e0b6f2a633a7afaa8cf4f2480b/intermediate/docker-compose/compose-cheatsheet.md#basic-example)Basic example

```
version: '2'
services:
 web:
 build: .
 # build from Dockerfile
 context: ./Path
 dockerfile: Dockerfile
 ports:
 - "5000:5000"
 volumes:
 - .:/code
 redis:
 image: redis
```

## [](https://github.com/collabnix/dockerlabs/blob/ce94351e2ba629e0b6f2a633a7afaa8cf4f2480b/intermediate/docker-compose/compose-cheatsheet.md#commands)Commands

```
docker-compose start
docker-compose stop
docker-compose pause
docker-compose unpause
docker-compose ps
docker-compose up
docker-compose down
```

## [](https://github.com/collabnix/dockerlabs/blob/ce94351e2ba629e0b6f2a633a7afaa8cf4f2480b/intermediate/docker-compose/compose-cheatsheet.md#reference)Reference

## [](https://github.com/collabnix/dockerlabs/blob/ce94351e2ba629e0b6f2a633a7afaa8cf4f2480b/intermediate/docker-compose/compose-cheatsheet.md#building)Building

```
web:
# build from Dockerfile
 build: .
# build from custom Dockerfile
 build:
 context: ./dir
 dockerfile: Dockerfile.dev
# build from image
 image: ubuntu
 image: ubuntu:14.04
 image: tutum/influxdb
 image: example-registry:4000/postgresql
 image: a4bc65fd
```

## [](https://github.com/collabnix/dockerlabs/blob/ce94351e2ba629e0b6f2a633a7afaa8cf4f2480b/intermediate/docker-compose/compose-cheatsheet.md#ports)Ports

```
ports:
 - "3000"
 - "8000:80" # guest:host
# expose ports to linked services (not to host)
 expose: ["3000"]
 
## Commands
 command to execute
 command: bundle exec thin -p 3000
 command: [bundle, exec, thin, -p, 3000]
# override the entrypoint
 entrypoint: /app/start.sh
 entrypoint: [php, -d, vendor/bin/phpunit]
```

## [](https://github.com/collabnix/dockerlabs/blob/ce94351e2ba629e0b6f2a633a7afaa8cf4f2480b/intermediate/docker-compose/compose-cheatsheet.md#environment-variables)Environment variables

```
# environment vars
 environment:
 RACK_ENV: development
 environment:
 - RACK_ENV=development
# environment vars from file
 env_file: .env
 env_file: [.env, .development.env]
```

## [](https://github.com/collabnix/dockerlabs/blob/ce94351e2ba629e0b6f2a633a7afaa8cf4f2480b/intermediate/docker-compose/compose-cheatsheet.md#dependencies)Dependencies

```
# makes the `db` service available as the hostname `database`
# (implies depends_on)
 links:
 - db:database
 - redis
# make sure `db` is alive before starting
 depends_on:
 - db
```

## [](https://github.com/collabnix/dockerlabs/blob/ce94351e2ba629e0b6f2a633a7afaa8cf4f2480b/intermediate/docker-compose/compose-cheatsheet.md#other-options)Other options

```
# make this service extend another
 extends:
 file: common.yml # optional
 service: webapp
 volumes:
 - /var/lib/mysql
 - ./_data:/var/lib/mysql
```

## [](https://github.com/collabnix/dockerlabs/blob/ce94351e2ba629e0b6f2a633a7afaa8cf4f2480b/intermediate/docker-compose/compose-cheatsheet.md#advanced-features)Advanced features

## [](https://github.com/collabnix/dockerlabs/blob/ce94351e2ba629e0b6f2a633a7afaa8cf4f2480b/intermediate/docker-compose/compose-cheatsheet.md#labels)Labels

```
services:
 web:
 labels:
 com.example.description: "Accounting web app
```

## [](https://github.com/collabnix/dockerlabs/blob/ce94351e2ba629e0b6f2a633a7afaa8cf4f2480b/intermediate/docker-compose/compose-cheatsheet.md#dns-servers)DNS servers

```
services:
 web:
 dns: 8.8.8.8
 dns:
 - 8.8.8.8
 - 8.8.4.4
```

## [](https://github.com/collabnix/dockerlabs/blob/ce94351e2ba629e0b6f2a633a7afaa8cf4f2480b/intermediate/docker-compose/compose-cheatsheet.md#devices)Devices

```
services:
 web:
 devices:
 - "/dev/ttyUSB0:/dev/ttyUSB0"
```

## [](https://github.com/collabnix/dockerlabs/blob/ce94351e2ba629e0b6f2a633a7afaa8cf4f2480b/intermediate/docker-compose/compose-cheatsheet.md#external-links)External links

```
services:
 web:
 external_links:
 - redis_1
 - project_db_1:mysql
```

## [](https://github.com/collabnix/dockerlabs/blob/ce94351e2ba629e0b6f2a633a7afaa8cf4f2480b/intermediate/docker-compose/compose-cheatsheet.md#hosts)Hosts

```
services:
 web:
 extra_hosts:
 - "somehost:192.168.1.100"
```