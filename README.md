# Docker Lab instructions
Please note, this tutorial requires you to have Docker Community Edition installed on your computer.  Get it [here](https://www.docker.com/community-edition).
---
## 1. Docker Hello World
Please note: if it seems like a lot is being introduced right off the bat, it's just to expose you to it.  We will be exploring everything touched on here in greater details as the tutorial proceeds.

On the command line, type in:
```
docker run hello-world
```

You will see the following output:
```
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://cloud.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/engine/userguide/
 ```
 
The description of what happened is a pretty good summary.  But here are some more details:
1. The Docker client, invoked via the command line contacts the Docker daemon running on your system.
2. The Docker daemon pulls the "hello-world" image.
   * Docker is by default configured to search Docker Hub for images, as opposed to say, your organization's private repo.
   * In order to specify a tag for an image, you would append the tag name with a colon after the image name, such as
     ```docker run hello-world:linux```
     If you do not specify a tag, then Docker will treat it as if you requested the "latest" version of the image, as in
     ```docker run hello-world:latest```
     The "latest" tag is what Docker automatically tags images with when you do not explicitly provide one at build time.
     ```docker build my-container``` is equivalent to ```docker build my-container:latest```.  We will explore well-known problems with the "latest" tag in a bit.
3. The Docker daemon creates a new container based on the "hello-world" image, and starts it.  In fact, the ```docker run``` command is a shorthand for two commands: ```docker create``` (which instantiates the container from the image) and ```docker start <image>``` (which actually runs it).
4. The Docker daemon streams the output to the Docker client, which in turn sends it to your CLI.  By default, Docker will stream a container's output to your terminal.  You can ensure that Docker does not output to your terminal by using the -d or --detach flag, as in: ```docker run -d hello-world```.  You can think of the default as running a command in linux, outputting to the terminal vs. appending the command with an ampersand, which backgrounds the process and does not attach its output to your terminal.  i.e. ```top``` vs. ```top &```.
---
## 2. Docker Hub
Here is the URL for the Docker Hub page for the hello-world image (this page is its "repository"): https://hub.docker.com/_/hello-world/  -- It is worth familiarising yourself with the layout and content of Docker Hub pages.

![Docker Hub](/images/dockerhub.png)

1. I find the _Tags_ page to be a much better and clearer listing of a repository's tags.  Those in the _Full Description_ section are manually entered while those on the _Tags_ page come from Docker itself.
![Docker Hub Tags page](/images/dockerhubtags.png)
2. You are always provided with the command needed to pull an image, similar to how you can get a git repo's link to clone it locally.
3. The _Full Description_ will provide a lot of interesting details about images.  It is common for the repository's maintainers to provide links to the Dockerfile(s) used to create the image.  This is in keeping with the open-source nature of a lot of Docker work - you can see for yourself exactly what is and is not in an image.  The content and quality of the _Full Description_ is going to vary by the maintainer(s).  My experience has been that "official" repos for products, e.g. Docker-related repos, Redis, Wordpress, etc. tend to have the best documentation.
---
## 3. A Taste of the Power of Docker
1. Run the following command to clone the specified git repo to your local host:
```git clone https://github.com/spkane/wearebigchill.git --config core.autocrlf=input```
2. [Click here](/outputs/wearebigchill-docker-build-output.txt) for the output that you ought to be seeing from this command.
   * Here is a subset of the output:
     ```Step 1/13 : FROM fedora:22
        ---> 01a9fe974dba
        
        ...
        
        Step 11/13 : ENV THEME 1
        ---> Running in 485f1c9d017c
        Removing intermediate container 485f1c9d017c
        ---> b3d9e3838d1
        Step 12/13 : EXPOSE 80
        ---> Running in 7b0a55e4a76b
        Removing intermediate container 7b0a55e4a76b
        ---> 3a348f8ca063
      ```
   * Some interesting things to note on what is going on here:
     1. Docker builds up images in an incremental, back and forth fashion:
        1. An image is instantiated as an "intermediate container".
           * Step 12/13 instantiates the image layer ```b3d9e3838d1``` from Step 11/13.  This intermediate container has an id of ```7b0a55e4a76b```.
        2. A command is executed in that container to change its state in some desired way (changing contents, variables, etc.).
           * In Step 12/13 this command is ```EXPOSE 80```, which tells Docker to expose port 80 on the app to the host OS (More on this in a bit).
        3. A snapshot of the container after step 2. is captured as a new image.  
           * After executing the EXPOSE 80 command in the intermediate container, Docker captures this new state as a new image container: ```---> 3a348f8ca063```
        4. Go back to Step 1 until you have reached the end of your Docker commands.
           
     2. The first image layer we see is 01a9fe974dba as part of Step 1/13.  This layer is populated by the FROM fedora:22 command, which means "Start with fedora:22 as my base image to build upon."  You always have to start with a FROM command, to specify what to start from.
     3. Docker will remove intermediate containers, e.g. ```Removing intermediate container 7b0a55e4a76b```.  It is not clear to me at this time why Docker does not display this message for ```ADD``` commands.
         
