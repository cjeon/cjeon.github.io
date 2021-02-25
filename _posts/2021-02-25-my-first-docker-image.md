---
layout: post
title: "My first docker image"
---
In this article, I will introduce

1. Why I decided to learn Docker - the problem I tried to solve
2. How I wrote the docker file
3. How to optimize docker build cache
4. How to create and attach a volume to docker instance, and how to run the docker instance

## Introduction
I have always wanted to learn Docker. Many people and frameworks around me used it, which made me think I'm being left behind. I took some how-to tutorials, but I never had a chance to solve real-world problems until recently.

Recently I had a chance to use Docker: me (front-end engineer) and back-end engineers were doing a job together, and our codes needed to be mixed. Once some back-end job is done, some front-end should start and yield some result, and after that, depending on the result, some back-end job should start. 

But since no one party knows both front-end and back-end, we decided to containerize every step, and each party only needs to manage containers related to them. We will build container "blocks" that take some input and produces some output. Once our block building is complete, we will assemble them, and our project will be done without anyone having to know something out of their scope. And later, if some middle-step needs an update, we only need to update related container images. What an excellent co-op!

## Dockerfile
So below is my "cute" first docker file.

```
FROM cimg/python:3.9.1

WORKDIR /app
RUN sudo chmod 777 .
COPY requirements.txt requirements.txt 
RUN pip3 install -r requirements.txt
COPY . . 

CMD python3 firebase.py [args]
```
What it does is pretty simple.  Our base image is `cimg/python:3.9.1`. It's a simple image with built-in Python-related tools provided by CircleCI (we will primarily run this image on CircleCI).  Then we will create and enter `/app` and change the access permission to `777`.  Then we copy the `requirements.txt`, and we install dependencies. 

## Docker build cache optimization
Note that we execute these two lines before running `COPY . .`. I did this to make more use of cache during the build process. While our `requirements.txt` seldom changes, items in the directory (.) change in every build. If we used 
```
COPY . . 
RUN pip3 install -r requirements.txt
```
instead of
```
COPY requirements.txt requirements.txt 
RUN pip3 install -r requirements.txt
COPY . . 
```
What will happen is,
1. We will first copy all items in the directory - by doing this, a new image will be made since things in the current directory are changed.
2. Since the previous image is new, `RUN pip3 install -r requirements.txt` has no cache and will have to be executed. This will cost a long time (more than a minute) every build.

Instead, if we used the current version, what will happen is,
1. We will first copy `requirements.txt` - since the contents of `requirements.txt` are not changed, we will use cached images.
2. Since nothing changed so far, `RUN pip3 install -r requirements.txt` will not be executed, and the Docker will use the cached image, saving us more than a minute.

This cache optimization might not look significant, but if you are a docker newbie or have to build an image many times to complete one, saving a minute every build can be very helpful. 

## CMD and RUN
Back to the docker file. Finally, we run `CMD python3 firebase.py [args]`. The image will run `python3 firebase.py [args]` once someone starts the image without any command to execute. I understood this as a default command.

`RUN` and `CMD`'s difference is (they are totally different): `RUN` is executed during the build process to build an image, and `CMD` is executed on the built image after an image is built.

## Building an image using the docker file
I built my image using the following command
```
docker build -t name .
```
It will create an image using the current directory's `Dockerfile` and name it `name:latest`. The default tag `latest` is used since I did not provide the tag.

Then, by running the below command, I can confirm that my image is built well 👍.
```
> docker image ls -a
REPOSITORY                  TAG                IMAGE ID       CREATED              SIZE
name   latest             f0b5b98b9a71   13 seconds ago       2.37GB
```

## Running an docker instance using the image
I ran my image with below command
```
> docker run \
-e var1=$var1 \
-e var2=var2 \
--mount 'type=volume,src=name-volume,dst=/app/results' \
-it name:latest
```
This will
1. Set environment variables. `var1` will be set as the current environment's `$var1`, `var2` will be set as string `var2`.
2. Mount volume `name-volume` to the instance, to path `/app/results`. By doing this, we can access the docker instance's outputs.
3. The flag `-i` is for interactive, `-t` is for pseudo-terminal (so that we could send inputs and receive outputs).
4. Then we specify the image name and tag. And we don't give any command to run, to use default `CMD`.

We can create a volume by executing `docker volume create name-volume`, and we can view contents of the volume by multiple ways. I use VS code to view them. See https://code.visualstudio.com/docs/remote/containers#_inspecting-volumes for details.

We can create a volume by executing `docker volume create name-volume`, and we can view contents of the volume in multiple ways. I use VS code to view them. See https://code.visualstudio.com/docs/remote/containers#_inspecting-volumes for details.

## Outro
That's it! We just have built a docker image and ran an instance of it. It took me several days to learn it, but it was amazingly fun and totally worth it. I hope this article can be helpful to someone 😁.