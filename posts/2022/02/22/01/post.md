# Quick top 5 `docker` cli shortcuts

Difficulty level: **low**

Containerization is an excellent way of applications isolation and Docker is one
the most popular tools which allow us to do that.

In this short post i'm trying to put handy cli shortcuts which i use almost
every day.

Alternatives to docker are: Podman, LXD, Containerd, RunC, Buildah, BuildKit,
Kaniko.

The first 3 sounded familiar to me, though the last 4 is something to take
a look and consider.

-more-

## Build image

Build an _image_ from Dockerfile:

```
docker build --no-cache=true -t <image-name>:latest .
```

--or-- with passed build variable `staging` (use `--build-arg`)

```
docker build --build-arg working_branch=staging --no-cache=true -t <image-name>:latest .
```

--or-- if you have multiple Dockerfiles within the single code repo
(monorepository?) - let's say all Dockerfile's are in separate folders (`api` or
`spa` in this example):

```
docker build -t <image-name>:latest -f dockerfiles/api/Dockerfile .
docker build -t <image-name>:latest -f dockerfiles/spa/Dockerfile .
```

 #hashtag1 #hashtag2