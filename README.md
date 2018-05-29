# Docker Lab
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
2. Change into the new folder: ```cd wearebigchill```
3. Build the Docker image: ```docker build .```
   [Click here](/outputs/wearebigchill-docker-build-output.txt) for the output that you ought to be seeing from this command.
   The output will end with the following line: ```Successfully built 49aee06e8e38```. Your image id will differ from ```49aee06e8e38```.  You will need this id for the next command.
4. Run the docker image: ```docker run -p 8090:80 <image ID from previous step>```.
5. Go to [http://localhost:8090](http://localhost:8090) in your browser and try the game out for a spin.  Try not to mind the very suspect physics.  
6. Admire how dead simple it actually was to start up a web server that serves up this "game".  
7. Go back to your terminal and hit ```ctrl+c``` to stop the container.
8. Restart the container, this time specifying an environment variable (more on this in a bit): ```docker run -p 8090:80 -e "THEME=2" <same image ID>```
9. Refresh [http://localhost:8090](http://localhost:8090) and observe how it has changed.  You may need to open it in a private browsing window instead to avoid the browser cache loading up the previous version of the app.

Bottom line: Docker makes it incredibly easy to distribute software that "just runs" on any system (that has Docker).  It is also extremely simple to configure the behaviour of your apps for different environments and situations.  There are other ways to modify a container's configuration besides the setting of environment variables via the -e command.

---
## 4. An Aside on the Creation of Images
This section explores some (in my opinion) interesting details about how Docker goes about creating images.  It is not strictly necessary in order to use Docker, but I think it helps understand what images are and how they are formed.
Here is a subset of the output from the ```docker build .``` command in the ```wearebigchill``` project folder:
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
So what is going on here?  Some interesting things to note:
1. Docker builds up images in an incremental, back and forth fashion:
   1. An image is instantiated as an "intermediate container".
      * Step 12/13 instantiates the image layer ```b3d9e3838d1``` from Step 11/13.  This intermediate container has an id of ```7b0a55e4a76b```.
   2. A command is executed in that container to change its state in some desired way (changing contents, variables, etc.).
      * In Step 12/13 this command is ```EXPOSE 80```, which tells Docker to expose port 80 on the app to the host OS (More on this in a bit).
   3. A snapshot of the container after step b is captured as a new image.  
      * After executing the EXPOSE 80 command in the intermediate container, Docker captures this new state as a new image layer: ```---> 3a348f8ca063```
   4. Go back to step a until you have reached the end of your Docker commands.
2. The first image layer we see is 01a9fe974dba as part of Step 1/13.  This layer is populated by the FROM fedora:22 command, which means "Start with fedora:22 as my base image to build upon."  You always have to start with a FROM command, to specify what to start from.  If you actually want to start an entirely new image, you would specify FROM scratch, which is an empty image.  For what it is worth, it is exceedingly unlikely that we would ever need to do this -- there is always going to be an image that has done much of the groundwork for us to build upon.
3. Docker will remove intermediate containers, e.g. ```Removing intermediate container 7b0a55e4a76b```.  It is not clear to me at this time why Docker does not display this message for ```ADD``` commands.
4. The ID for the final container -- what you actually want out of this whole process -- is displayed at the end: ```Successfully built 49aee06e8e38```.  We will learn about how to give images more human-friendly names later.

---
## 5. So Where Do Images Come From Anyway?

Dockerfiles!

If you open the Dockerfile inside of the ```wearebigchill``` directory, it looks like the following:
```FROM fedora:22
   
   RUN dnf install -y httpd
   RUN mkdir /var/www/html/res && \
       mkdir /theme1 && \
       mkdir /theme2
   ADD httpd.conf /etc/httpd/conf/httpd.conf
   ADD publish/html5/theme1/* /theme1/
   ADD publish/html5/theme2/* /theme2/
   ADD publish/html5/game.min.js /var/www/html/
   ADD publish/html5/index.html /var/www/html/
   ADD publish/html5/project.json /var/www/html/
   ADD start.sh /
   ENV THEME 1

   EXPOSE 80

   CMD ["/start.sh"]
```
If you look back at the output of the docker build process in the wearebigchill folder [here](/outputs/wearebigchill-docker-build-output.txt) you should see some striking similarities between the Dockerfile and that output.

### An aside
Every Docker command (RUN, ADD, ENV, etc.) results in an additional layer being added to your image.  Certainly at our level of Docker it's not as critical to be hyper vigilant, but in general it is a best practice to reduce the number of commands/layers that go into your Dockerfiles/images.  One way to do this can be observed above where the second RUN command actually uses linux's ability to chain different command line invocations into the same command.  That command COULD be:
```
RUN mkdir /var/www/html/res 
RUN mkdir /theme1
RUN mkdir /theme2
```
...but this is discouraged because a new layer will be added for each separate RUN command and this makes your resulting images larger.  This in turn is undesirable because it means more bandwidth is required to download and deploy your Docker images to servers or anywhere else.

- [ ] Add in information about multi-stage builds here?

### Back to our regularly scheduled programming...
Dockerfiles consist of commands and the content for said commands, in the format ```<COMMAND> <CONTENT>```
In this particular file, there are 6 different types of commands:

1. ```FROM <image>```
This instructs Docker what image to use as a start.  Depending on your needs, you could, for instance, start with different flavours    of linux (FROM [alpine](https://hub.docker.com/_/alpine/), FROM [centos](https://hub.docker.com/_/centos/), FROM [debian](https://hub.docker.com/_/debian/), etc.), or start with an image that already has some kind of runtime or technology that you want your container to feature (FROM [openjdk](https://hub.docker.com/_/openjdk/), FROM [python](https://hub.docker.com/_/python/), FROM [ruby](https://hub.docker.com/_/ruby/), etc.).  Alpine linux (FROM alpine) is generally a safe bet for a generic image to start from.  It is a very slimmed down distribution of linux intended for use in Docker containers.  A best practice is to specify a version tag on an image to build from, e.g. ```FROM alpine:3.7``` because otherwise you are at the mercy of whatever the latest tag gets updated to point to, which might change something unexpectedly.

2. ```RUN <linux command>``` - This runs a linux command, the results of which then become part of your image.  So ```RUN mkdir /var/www/html/res``` will create the specified directory within the container.  ```RUN apt-get install vim``` would install the greatest text editor in the universe onto a Ubuntu-based image.  As noted in the aside above, keep an eye out for the opportunity to combine RUN commands to reduce the number of layers required to build your image and reduce its size.

3. ```ADD <file on host> <location in container to put it>``` - The ADD command lets you copy files from your host OS into the container.  This is really useful for getting assets, configuration or properties files, shell scripts, or indeed your artifact itself (e.g. your Spring Boot JAR) into the container.  For example, ```ADD money-maker.jar /``` would add this wonderful app to the root folder of the container. 

4. ```ENV <environment variable> <value>``` - This command sets an environment variable on the container to the specified value.  For example, ```ENV GITHUB_REPO https://github.com/ghostandgrey/docker-lab``` It should be noted that this can be overridden when running the container.  So for instance, even with the GITHUB_REPO being set to that, we could override it using the -e flag: ```docker run -e "GITHUB_REPO=https://github.com/evil-impostor-repo"```.  This is exactly what we did earlier with the Docker game.  The image already had an environment variable specifying that the first theme should be used, but we overrode it to use the second one.

5. ```EXPOSE <port number>``` - The EXPOSE command is actually largely intended for documentation purposes - to communicate to users of the images what ports need to be mapped.  Because of namespacing, containers have their entire own series of ports which are not by default accessible from the host OS or beyond.  If you are running Jenkins in a container on port 8080, then hitting http://localhost:8080 will not hit the container.  We'll see how to set this sort of thing up in a bit.  We will also see that the EXPOSE command can in fact be used to get ports mapped to your container, but that this is not a good idea.

6. ```CMD ["<linux command>"]``` - This runs a linux command, like the RUN command.  However, this linux command is executed when the container is started.  So it could be the command to start your app.  For a lot of the linux images, e.g. alpine, this command will typically be ```/bin/sh```, meaning that the container is just running a shell at start-up. 

So let's try to read the Dockerfile for wearebigchill again in light of our understanding of these commands.

```FROM fedora:22
   
   RUN dnf install -y httpd                      Use fedora's package manager to install httpd without requiring input.
   RUN mkdir /var/www/html/res && \              Create these three folders 
       mkdir /theme1 && \
       mkdir /theme2
   ADD httpd.conf /etc/httpd/conf/httpd.conf     Copy our httpd.conf file into the appropriate place in the container for httpd to use.
   ADD publish/html5/theme1/* /theme1/           Copy all of our theme1 assets into the container.
   ADD publish/html5/theme2/* /theme2/           Copy all of our theme2 assets into the container.
   ADD publish/html5/game.min.js /var/www/html/  Copy the JavaScript for the game into the container.
   ADD publish/html5/index.html /var/www/html/   Copy the HTML for the game page into the container.
   ADD publish/html5/project.json /var/www/html/ Copy the game's configuration settings into the container.
   ADD start.sh /                                Copy the script responsible for starting the httpd server into the container.
   ENV THEME 1                                   Set the THEME environment variable to a value of 1.

   EXPOSE 80                                     Open up port 80 so that the host OS can see it.

   CMD ["/start.sh"]                             Set the container to execute the start.sh script when it starts up.
```
Hopefully it makes more sense now how the Dockerfile defines an image, and leads to your running container!  Hopefully you are starting to get a sense of how you could combine these (and oh so many other Dockerfile commands) to create a variety of different containers to suit your purposes.

If you for some reason don't have access to the Dockerfile that created an image, you can still get a sense for how it was crafted using the ```docker history <image ID>``` command.  You will not see explicitly what files were copied into the image.  For instance:
```docker history f61bd71f06d7
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
f61bd71f06d7        6 days ago          /bin/sh -c #(nop)  CMD ["/outyet" "-version"…   0B
3eeb8619cd11        6 days ago          /bin/sh -c #(nop)  EXPOSE 18088                 0B
5de5dcc8d294        6 days ago          /bin/sh -c #(nop) COPY file:20879a19eead23ea…   5.64MB
afd7c486820e        6 days ago          /bin/sh -c #(nop) WORKDIR /                     0B
ea99ba7b96d3        6 days ago          /bin/sh -c apk --no-cache add ca-certificates   555kB
3fd9065eaf02        4 months ago        /bin/sh -c #(nop)  CMD ["/bin/sh"]              0B
<missing>           4 months ago        /bin/sh -c #(nop) ADD file:093f0723fa46f6cdb…   4.15MB
```

---
## 6. Creating Images

Download the jar file located [here](https://www.dropbox.com/s/cmh87m1rvo39nvz/mtdan-1.0-SNAPSHOT.jar?dl=0) to a directory other than the wearebigchill one we've been in this whole time.  Let's pretend that it is your project's deployable artifact, and not the steaming pile of garbage that it is. (FYI - It is safe but I assure you it is garbage.  Mike and I hacked this thing together last night for illustrative purposes.).  It is a Spring Boot jar.  If you like, you can run it to see what it does (```java -jar mtdan-1.0-SNAPSHOT.jar```).  You can hit it at [localhost:8080](http://localhost:8080).  It should say  "Who are you?"  This is because it looks for a name under the YOUR_NAME environment variable, which I'm thinking is a safe bet to not be defined on anybody's computer.  Let's get this thing running in Docker!

Step 1: The Dockerfile.
1. The first thing we need is to have an image with Java in it.  We COULD take a linux image and install Java into it ourselves, but why reinvent the wheel?  Go to [hub.docker.com](https://hub.docker.com) and find a suitable image to start with.
<details>
 <summary>If nothing is jumping out at you in the first few pages, click on this line for our recommendation</summary>
 openjdk (Don't worry about a specific tag/version for this exercise.)
</details>
  
Use your favourite editor to create a file in the same directory as the downloaded jar, naming this file ```Dockerfile``` (no extension).
The first line is going to be ```FROM <base image you have decided upon>```.

2. Next, we need to actually get the jar itself into the image.  This can be done with a command very similar to the ADD command - the COPY command.  It has the same syntax: ```COPY <local file> <location on container>```  Place it into root.  COPY differs from ADD insofar as it is a straight copy.  ADD will do some additional processing of the file depending on what it is.  For instance, ```ADD some-tar-file.tar /``` would copy the tar file to the image, and then extract it automatically.  ```COPY some-tar-file.tar /``` will copy the file and do nothing else with it.  It is a best practice to use COPY rather than ADD for when you are merely wanting to get a file into the image.

<details>
 <summary>Click here if you want to see what this line should look like</summary>
 COPY mtdan-1.0-SNAPSHOT.jar /
</details>
    
3. Finally, we want the spring boot jar with its embedded Tomcat server to start whenever the container starts.  This is achieved with the CMD command.  There are a few different syntaxes for this, but the most common one I've seen is called "exec form".  More details on it can be found [here](https://docs.docker.com/engine/reference/builder/#cmd).  It requires that whatever linux command you want to run is an array of Strings, delineated by spaces.  So for instance, if you wanted to run ```ps -aux```, this would actually be ```["ps", "-aux"].  In light of this, try and figure out how you would input the ```java -jar /mtdan-1.0-SNAPSHOT.jar``` command.  
<details>
 <summary>The answer is here</summary>
 CMD ["java", "-jar", "/mtdan-1.0-SNAPSHOT.jar"]
</details>

<details>
 <summary>Here is what your Dockerfile should currently look like</summary>
 FROM openjdk
 COPY mtdan-1.0-SNAPSHOT.jar /
 CMD ["java", "-jar", "/mtdan-1.0-SNAPSHOT.jar"]
</details>



---
## 7. Exploring Containers

### Listing Containers
You can list all running containers using the ```docker ps``` command (think ps, like the linux command for listing running processes).
If the wearebigchill container is not running, run it now, using the ```docker run -p 8090:80 <image ID>``` command.  If you do not have the image ID handy, you can list all images you currently have downloaded on your system with the ```docker images``` command:
```
$docker images
REPOSITORY           TAG                                        IMAGE ID            CREATED             SIZE
<none>               <none>                                     49aee06e8e38        26 hours ago        355MB
```
So after ensuring that the container is running, type in ```docker ps```.  You should see something like the following output:
```
docker ps
CONTAINER ID   IMAGE         COMMAND      CREATED             STATUS              PORTS                     NAMES
c6a4cefa06ad   49aee06e8e38  "/start.sh"  24 hours ago        Up 21 hours         0.0.0.0:8090->80/tcp      loving_rosalind
```

### Container Names
```loving_rosalind```??  If you do not specify a name for a docker container (we'll see how to do this in a bit) then Docker will come up with a name for it, combining an adjective and a prominent technological/scientific pioneer with an underscore.  You can actually look at the go code [here](https://github.com/moby/moby/blob/master/pkg/namesgenerator/names-generator.go) that generates these names.  Whoever wrote it apparently thinks Steve Wozniak is very interesting.

If you prefer your containers to have more meaningful names, then you can specify one when you first instantiate the container with the --name <container_name> flag.  So for instance, if we were to invoke ```docker run --name hello-world hello-world```, it would create an instance of the hello-world image and also call it "hello-world".  Naming containers is a good practice.  Not only does it make it easier to identify and differentiate your containers, you can also use their names to refer to them in commands.  So you could say ```docker inspect goodbye_world``` rather than ```docker inspect 151dae5a...blah blah blah```.

### Listing Containers, Continued
We saw that you can list all running containers with the ```docker ps``` command.  When containers stop running, they are not by default deleted.  You can list ALL containers, whether running or stopped with the -a flag: ```docker ps -a```.  You might be surprised how many you accrue from time to time.  Try this, and note that you should have multiple instances of the image we ran earlier for that Docker game.  How does one remove unnecessary Docker containers?  I'm glad you asked!

### Deleting Containers
There are three primary ways to delete containers:
1. Once a container has stopped, invoking ```docker rm <container name or ID>``` e.g. ```docker rm hello-world```.


- logs
- sshing in
- removing containers
Docker allows you to execute arbitrary commands inside of already-running containers.  This can be useful for debugging containers, but is generally frowned upon for production purposes.  It is considered best practice to have your container in its final state 

---
## 7. Volumes
- Demonstrate persistence
- https://hub.docker.com/r/binocarlos/hello-increment/

---
## 8. Ports
- EXPOSE command in more detail
- host vs. container ports
- ephemeral ports when none are specified

---
## 9. Networking
- Find good, simple example of networking containers.'
- https://docs.docker.com/network/network-tutorial-standalone/


---
## 10. Docker Compose
- https://docs.docker.com/compose/wordpress/#define-the-project


