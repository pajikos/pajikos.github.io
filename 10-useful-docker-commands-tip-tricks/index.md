# 10 Useful Docker Commands - Tips and Tricks


## Set HTTP proxy in your Dockerfile

Your Dockerfile usually starts most probably like this:

```dockerfile
FROM tifayuki/java:8
MAINTAINER ...
RUN apt-get update \
wget download.java.net/glassfish/4.0/release/glassfish-4.0.zip \
...
```

At first, you are apt-getting some missing applications and preparing your environment. Sometimes some applications need to be downloaded from the internet. You may encounter a problem when you are behind a proxy. Fortunately, you can use an `[ENV](https://docs.docker.com/reference/builder/)` command to set a HTTP/HTTPS proxy address, which will be in the environment of all "descendent" Dockerfile commands. So the updated Dockerfile example:

```dockerfile
FROM tifayuki/java:8
MAINTAINER ...
ENV http_proxy http://server:port
ENV https_proxy http://server:port
#... some other online commands
```

##  Mount one folder into multiple locations

The Docker architecture is based on [union mount](https://en.wikipedia.org/wiki/Union_mount), namely [UnionFS](https://en.wikipedia.org/wiki/UnionFS "UnionFS"), which gives you an ability to mount volumes (folders) to any location in your image, let's see the following example (using this image [tutum/glassfish](https://github.com/tutumcloud/glassfish)):

```bash
docker run -d -p 4848:4848 -p 8080:8080 -p 8181:8181 -v /data/output/gf-logs:/opt/glassfish4/glassfish/domains/domain1/logs -v /data/output/gf-logs:/var/log tutum/glassfish
```

See both of `-v` params, they are mounting the same folder (/data/output/gf-logs)  into exactly two existing locations (Glassfish server log and /var/log/).

Finally, you have only one folder with all logs from your Glassfish.

## Long-running worker process

If you are new to the docker environment, you may be surprised, that Docker containers only run as long as the command you specify is active. E.g. this ubuntu instance immediately exits after echoing "Hello World":

```bash
docker run ubuntu /bin/echo 'Hello world'
```

If you want to create a container that runs as a daemon (like most of the applications), you need to specify a long-running command, e.g. the command that echoes "hello world" forever:

```bash
docker run -d ubuntu /bin/sh -c "while true; do echo hello world; sleep 1; done"
```

## Lists of all existing containers (not only running)

```bash
$ docker ps -a
```

## Stop all running containers

```bash
docker stop $(docker ps -a -q)
```

## Delete all existing containers

```bash
docker rm $(docker ps -a -q)
```

If some containers are still running as a daemon, use `-f` (force) param immediately after rm command.

## Delete all existing images

```bash
docker rmi $(docker images -q -a)
```

## Attach to a running container

[Docker Command Line API](https://docs.docker.com/reference/commandline/attach/) has a special command to attach to a running container. This command is not applicable in all cases, especially if a docker container has been already started using `/bin/bash` command, e.g. already known long running process:

```bash
docker run -d  ubuntu /bin/sh -c "while true; do echo hello world; sleep 1; done"
```

Trying to use `attach` command does not start a new instance of a shell, instead of that, you will see an ongoing output of the first command, i.e."hello world". Since Docker version 1.3 you are able to create a new instance of a shell using exec command:

```bash
sudo docker exec -i -t 878978547 bash
```

## Using Docker logs command

In a case of running a Docker container as a daemon (`docker run -d` ...) it may be useful to know what appears on the console output of the running container.

The `docker logs` retrieves logs present at the time of execution.  You need to run this command repeatedly to see the actual output. This is not good enough.

Fortunately, it exists a helpful param `-f, --follow,` which follows log output and continues streaming the new output from the container’s `STDOUT` and `STDERR`.:

```bash
docker logs -f 878978547
```

**NOTE**: this command is available only for containers with `json-file` logging driver.

## Replace CMD command in a Dockerfile

You should know, that it exists two similar commands in a Dockerfile, i.e. `CMD` and `RUN`.

`RUN` actually runs a command and commits the result at build time.

The `CMD` instruction should be used in a Dockerfile only once to run the software contained by your image at runtime. (mostly the last command in a Dockerfile).

> The main purpose of a CMD is to provide defaults for an executing container. ([source](https://docs.docker.com/reference/builder/))

You are able to replace a default CMD command from a Dockerfile, e.g. you need to customize this command:

```bash
docker run myImage "{CMD} -extraParam"
```

Again, CMD instruction in a Dockerfile is overwritten with your command from the command line.

