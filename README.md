# Docker-task

Scinario-1.Build a sample web application and run it inside a docker container.

For this need a webserver with sample web-application, I have downloaded a sample web-page(one-page1) template to run in a nginix container

Steps.

Download one-page1 directory and create a Docker file like below & import a ngnix image and copy the one-page directory to default nginix document root

FROM nginx

COPY . /usr/share/nginx/

-------------------------------------------------------------------
Build the image using the command 

docker build -t aupe/webserver-test-v1 .

Sending build context to Docker daemon  3.415MB
Step 1/2 : FROM nginx
 ---> b8efb18f159b
Step 2/2 : COPY . /usr/share/nginx/
 ---> 99af67e3e761
Removing intermediate container adb5689edbcc
Successfully built 99af67e3e761
Successfully tagged aupe/webserver-test-v1:latest

---------------------------------------------------------------------

Pushing the image to Dockerhub so that it can be used from anywhere 

docker push aupe/webserver-test-v1
The push refers to a repository [docker.io/aupe/webserver-test-v1]
3ab35ef3455a: Pushed 
af5bd3938f60: Pushed 
29f11c413898: Pushed 
eb78099fbf7f: Pushed 

-------------------------------------------------------------------------

Run the container with a port assoiated to host machine so that we can verify site is working.

Dinkan one-page1 # docker run --name webapp-test -d -p 8888:80 aupe/webserver-test-v1
c89b918dcef30a03f1ea3720b9f96d33772b200fc3e6b0d1585d697275764e5f
Dinkan one-page1 # docker ps 
CONTAINER ID        IMAGE                    COMMAND                  CREATED             STATUS              PORTS                  NAMES
c89b918dcef3        aupe/webserver-test-v1   "nginx -g 'daemon ..."   5 seconds ago       Up 3 seconds        0.0.0.0:8888->80/tcp   webapp-test

--------------------------------------------------------------------------------

================================================================================

Scinario-2. Create multiple instances of the application using docker compose.

use the docker-compose file in this repo. 

haproxy:
  
  image: tutum/haproxy
  
  links:
  
    - net
  
  ports:
    
    - "5004:80"

net:

  image: aupe/webserver-test-v1

  ports:

    - "80"

---------------------------------------------------------------------------------


Placed one Haproxy image as Load balencer which is already available in Dockerhub.

Scale using the command >> docker-compose up -d --scale net=5

--------------------------------------------------------------------

Scinario-3. An option to deploy changes on all instances without any downtime overall. (a script would do)

This can be done by Docker rolling update option with delay & parallelism switches.

First we need to create a service using the image

 docker swarm init && docker service create --name web-app aupe/webserver-test-v1 

 we can inspect the service by using the command >> docker service inspect --pretty web-app

Suppose there is change in the web-application and we have built a v2 image of the same, ie >> aupe/webserver-test-v2

We can deploy using the below command 

docker service update --update-delay=10s --update-parallelism=1 --image aupe/webserver-test-v2 web-app

Since we have Load balencer is already in place it will redirect the requestes to both v1 and v2 containers resulting in a different ouput based on the update delay but there won't be any downtime.

We can also scale while updating the service with replica option >> docker service update --replicas=6 web-app

---------------------------------------------------------------------------------------------------------
