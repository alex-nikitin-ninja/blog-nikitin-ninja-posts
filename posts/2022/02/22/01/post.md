# Quick top 5 `docker` cli shortcuts

Difficulty level: **low**

Containerization is an excellent way for applications isolation and Docker is
one the most popular tools which allow us to do that.

In this short post i'm trying to put handy cli shortcuts which i use almost
every day.

Alternatives to docker are: Podman, LXD, Containerd, RunC, Buildah, BuildKit,
Kaniko.

The first 3 sounded familiar to me, though the last 4 is something to take
a look and consider.

-more-

## Build image

Build an _image_ from Dockerfile:

_(with `--no-cache` option)_
```
$ docker build --no-cache=true -t <image-name>:latest .
```

--or-- with passed build variable `staging` (use `--build-arg`)

```
$ docker build --build-arg working_branch=staging --no-cache=true -t <image-name>:latest .
```

--or-- if you have multiple Dockerfiles within the single code repo
(monorepository?) - let's say all Dockerfile's are in separate folders (`api` or
`spa` in this example):

```
$ docker build -t <image-name>:latest -f dockerfiles/api/Dockerfile .
docker build -t <image-name>:latest -f dockerfiles/spa/Dockerfile .
```


## Remove _containers_ or _images_

**Remove exited _containers_**

Handy if you run plenty of containers without `--rm` (`docker run --rm ...`)
option or if you had a failed build.

```
$ docker rm $(docker ps -a -q -f status=exited)
```

**Remove unused _images_**

Handy if you had  a build which failed or made another image revision and older
containers are no longer in use.

```
$ docker rmi -f $(docker images | grep "<none>" | awk "{print \$3}")
```

## List _containers_ and _images_

**Images:**
```
$ docker images -a
```

**Containers:**
```
$ docker ps -a
```

## Connect to existing physical network

Create network which works as a bridge to existing physical network. New network
name in this example will be `pub_net`

Assuming network details are (usually sent by DHCP server, but can be set
manually):

|Syntax|Description|
|-|---|
|**Subnet** | 192.168.0.0/24 (or 192.168.0.1 - 192.168.0.255, or 192.168.0.1-255)|
|**Gateway** | 192.168.0.1|
|**Physical interface name**| wlo1|

Command to create a docker network interface:

```
$ docker network create -d ipvlan --subnet=192.168.0.0/24 --gateway=192.168.0.1 -o parent=wlo1 pub_net
```

Then run a container with that interface attached:
```
$ docker run --net=pub_net -it --rm ubuntu bash
```

## Volumes mapping
Bypass folders from host instance into container instance. Docker does not like
relative paths, so need to specify the **full** path.

```
$ docker run -v /path/to/host/folder:/path/to/container/folder -d <image name>
```


 #hashtag1 #hashtag2