# MIGRATION TO THE СLOUD WITH CONTAINERIZATION. PART 1 – DOCKER & DOCKER COMPOSE
> What is Docker? Docker is a platform designed to help developers build, share, and run container applications. Docker provides the ability to package and run an application in a loosely isolated environment called a container.

**Install Docker and prepare for migration to the Cloud**
First, we need to install Docker Engine, which is a client-server application that contains:
  -  A server with a long-running daemon process dockerd.
  -  APIs that specify interfaces that programs can use to talk to and instruct the Docker daemon.
  -  A command-line interface (CLI) client docker.

-----
<img width="443" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/d068721a-756f-4269-b31f-72ce6888b82e">

Step 1: Pull MySQL Docker Image from Docker Hub Registry: `sudo docker pull mysql:latest`
 - If you are interested in a particular version of MySQL, replace latest with the version number. Visit Docker Hub to check other tags
 - Verify the image: `sudo docker image ls`

-----
<img width="850" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/686305c0-788c-4c79-b487-66717d7dca06">

Step 2: Deploy the MySQL Container to your Docker Engine
  *  Once you have the image, move on to deploying a new MySQL container with: `docker run --name <container_name> -e MYSQL_ROOT_PASSWORD=<my-secret-pw> -d mysql:latest`
      -  Replace <container_name> with the name of your choice. If you do not provide a name, Docker will generate a random one
      -  The -d option instructs Docker to run the container as a service in the background
      -  Replace <my-secret-pw> with your chosen password
      -  In the command above, we used the latest version tag. This tag may differ according to the image you downloaded
  *  Then, check to see if the MySQL container is running: Assuming the container name specified is MySQL: `docker ps -a `
-----
<img width="1053" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/185afbfb-afe6-4682-a78d-5b16cb64555a">


### CONNECTING TO THE MYSQL DOCKER CONTAINER

Step 3: Connecting to the MySQL Docker Container
We can either connect directly to the container running the MySQL server or use a second container as a MySQL client.
**Approach 1**
*Connecting directly to the container running the MySQL server:*
-----
Two approaches to connect to the container as shown in the image are:
  1. Run the command to enter the container shell first, then login to the mysql server when you are already logged in to the container shell. Enter password when prompted. It has to be the passwoord we set earlier: `sudo docker exec -it mysqldb bash`
  2. Run the entire command on one line and select mysql as the shell you want to log in directly to. Also enter the password when prompted: `sudo docker exec -it mysqldb mysql -p` . This can also work to specify the user: `sudo docker exec -it mysqldb mysql -p -u root`
<img width="808" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/d7bc3cd2-e7d2-4296-aa96-1171b3d66fac">

-----
Change the server root password to protect your database. Exit the the shell with exit command
**Flags used**
  -  `exec` used to execute a command from bash itself
  -  `-it` makes the execution interactive and allocate a pseudo-TTY
  -  `bash` this is a unix shell and its used as an entry-point to interact with our container
  -  `mysqldb` is the container name used earlier.  The second mysql in the command “docker exec -it mysqldb mysql -u root -p” serves as the entry point to interact with mysql container just like bash or sh
  -  `-u` mysql username
  -  `p` mysql password

**Approach 2**
*At this stage you are now able to create a docker container but we will need to add a network. So, stop and remove the previous mysql docker container.*
```
sudo docker ps -a
sudo docker stop mysqldb 
sudo docker rm mysqldb or <container ID> 04a34f46fb98
```
-----
<img width="1129" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/3bee0252-4581-4394-9d32-f1cb178b70a7">
-----

**First, create a network
For clarity’s sake, we will create a network with a subnet dedicated to our project and use it for both MySQL and the application so that they can connect: `docker network create --subnet=172.18.0.0/24 tooling_app_network`

-----
<img width="1129" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/633e7c01-4664-4d9f-b447-b77bed74a822">

*Run the MySQL Server container using the created network.*
-  First, let us create an environment variable to store the root password: `export $MYSQL_PW=<value>`
-  Verify the environment variable is created: `echo $MYSQL_PW`
-  Then, pull the image and run the container, all in one command like below: `sudo docker run --network tooling_app_network -h mysqldbhost --name=mysqldb -e MYSQL_ROOT_PASSWORD=$MYSQL_PW  -d mysql:latest`

**Flags used**
  -  `-d` runs the container in detached mode
  -  `--network` connects a container to a network
  -  `-h` specifies a hostname

If the image is not found locally, it will be downloaded from the registry.
Verify the container is running: `sudo docker ps -a`

-----
<img width="1349" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/49ac1ba5-4e68-43f1-9754-e60671ec7c90">


> It is best practice not to connect to the MySQL server remotely using the root user. Therefore, we will create an SQL script that will create a user we can use to connect remotely.

Create a file and name it `create_user.sql` and add the below code in the file: `CREATE USER 'jendy'@'%' IDENTIFIED WITH mysql_native_password BY 'jendyjasper'; GRANT ALL PRIVILEGES ON * . * TO 'jendy'@'%';`
*Run the script:*
*  Ensure you are in the directory create_user.sql file is located or declare a path
*  `sudo docker exec -i mysqldb mysql -uroot -p$MYSQL_PW < create_user.sql`
-----
<img width="1349" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/4581a6d5-05fa-4e4a-9bb6-91fad9ac2eb0">

verify you can login using the username and password: `sudo docker exec -it mysqldb mysql -ujendy -pjendyjasper`

-----
<img width="810" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/e3496e39-39b9-4040-b184-3fbf190f7da9">

### Connecting to the MySQL server from a second container running the MySQL client utility
The good thing about this approach is that you do not have to install any client tool on your laptop, and you do not need to connect directly to the container running the MySQL server.

**Flags used:**
  -  `--name` gives the container a name
  -  `-it` runs in interactive mode and Allocate a pseudo-TTY
  -  `-rm` automatically removes the container when it exits
  -  `--network` connects a container to a network
  -  `-h` a MySQL flag specifying the MySQL server Container hostname
  -  `-u` user created from the SQL script
  -  `admin` username-for-user-created-from-the-SQL-script-create_user.sql
  -  `-p` password specified for the user created from the SQL script

-----
<img width="1413" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/8609fc04-9ce0-4b4c-956d-548dc75a35fe">

### Prepare database schema
*Now you need to prepare a database schema so that the Tooling application can connect to it.*
-  Clone the Tooling-app repository: `git clone https://github.com/darey-devops/tooling.git`
-  On your terminal, export the location of the SQL file: `export tooling_db_schema=/tooling_db_schema.sql`
-  You can find the `tooling_db_schema.sql` in the `tooling/html/tooling_db_schema.sql` folder of cloned repo.
-  Verify that the path is exported: `echo $tooling_db_schema`

-----
<img width="1413" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/64cd783f-7a0a-45e0-a818-4e5a1f9d9157">

-----

**Use the SQL script to create the database and prepare the schema. With the docker exec command, you can execute a command in a running container.**
`sudo docker exec -i mysqldb mysql -uroot -p$MYSQL_PW < $tooling_db_schema`

**Update the .env file with connection details to the database**
The .env file is located in the html tooling/html/.env folder but not visible in terminal. you can use vi or nano:
```
sudo nano .env

MYSQL_IP=mysqldbhost
MYSQL_USER=jendy
MYSQL_PASS=jendyjasper
MYSQL_DBNAME=toolingdb
```
**Flags used:**
  -  `MYSQL_IP` mysql ip address “leave as mysqldbhost”
  -  `MYSQL_USER` mysql username for user export as environment variable
  -  `MYSQL_PASS` mysql password for the user exported as environment varaible
  -  `MYSQL_DBNAME` mysql databse name “toolingdb”

**Run the Tooling App**
> Containerization of an application starts with creation of a file with a special name – ‘Dockerfile’ (without any extensions). This can be considered as a ‘recipe’ or ‘instruction’ that tells Docker how to pack your application into a container.

**So, let us containerize our Tooling application; here is the plan:**
-  Make sure you have checked out your Tooling repo to your machine with Docker engine
-  First, we need to build the Docker image the tooling app will use. The Tooling repo you cloned above has a Dockerfile for this purpose. Explore it and make sure you understand the code inside it.
-  Run `docker build` command
-  Launch the container with docker run
-  Try to access your application via port exposed from a container


**Let us begin:**
-  Ensure you are inside the directory “tooling” that has the file Dockerfile and build your container : `sudo docker build -t tooling:0.0.1 .`
-  In the above command, we specified a parameter -t, so that the image can be tagged tooling"0.0.1 – Also, you have to notice the `.` at the end. This is important as that tells Docker to locate the Dockerfile in the current directory you are running the command. Otherwise, you would need to specify the absolute path to the Dockerfile.

**Run the container:**
`sudo docker run --network tooling_app_network -p 8085:80 -it tooling:0.0.1`

Let us observe those flags in the command.
-  We need to specify the `--network` flag so that both the Tooling app and the database can easily connect on the same virtual network we created earlier.
-  The `-p` flag is used to map the container port with the host port. Within the container, apache is the webserver running and, by default, it listens on port 80. You can confirm this with the CMD ["start-apache"] section of the Dockerfile. But we cannot directly use port 80 on our host machine because it is already in use. The workaround is to use another port that is not used by the host machine. In our case, port 8085 is free, so we can map that to port 80 running in the container.

-----

<img width="1434" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/5b6e549a-0b5b-43b0-9266-b46e97e454fc">

# PRACTICE TASK

## Part 1
1. Download php-todo repository from [here](https://github.com/darey-devops/php-todo)
2. Write a Dockerfile for the TODO app
3. Run both database and app on your laptop Docker Engine
4. Access the application from the browser

My Dockerfile content is shown below with comments of the things i did. 
```
#use the php image that comes with apache
FROM php:7.2-apache

#a multi stage build. This gets the composer image and copies the binary file to a different direct
COPY --from=composer:latest /usr/bin/composer /usr/local/bin/composer

#copy all the files from the root directory to the jenkins file
COPY . /var/www/html/


RUN apt-get -y update

RUN apt-get install zip -y
RUN apt-get install unzip -y
RUN docker-php-ext-install mysqli pdo pdo_mysql && docker-php-ext-enable pdo_mysql

RUN apt-get -y install git

RUN composer install

WORKDIR /var/www/html

#change the server name to localhost on the apache conf
RUN echo "ServerName localhost" >> /etc/apache2/apache2.conf

#migrate the database
RUN php artisan migrate

#copy the custom apache config file which grants access to the /var/www/html/public directory where the index file is
COPY apache-config.conf /etc/apache2/sites-available/000-default.conf
RUN a2enmod rewrite
RUN chown -R www-data:www-data /var/www/html
#CMD ["php", "artisan", "serve"]

#start apache
RUN apachectl start
```
-----

The built image result

<img width="750" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/85a3a157-edee-4131-9c4e-e1ac47a0e9ef">

-----

The running container result

<img width="750" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/f0c016e2-eab5-4b74-9c21-5d87aafb645d">

-----

The loaded todo webpage

<img width="1412" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/e9c1f070-2f4d-4142-8aae-82320f400969">


## Part 2
1. Create an account in Docker Hub
2. Create a new Docker Hub repository
3. Push the docker images from your PC to the repository

-----

DockerHub Repo

![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/4cd494fc-4003-4f90-86b1-973d436b4f51)


-----
Login to docker Hub on CLI

<img width="1369" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/8a140f67-ab4a-4649-a4f6-500337f4dac0">

-----

On The CLI, Use the `docker tag` command to give the `todo:v1.0` image a new name. Replace jendyjasper with your Docker ID.
`sudo docker tag todo:v1.0 jendyjasper/todo:v1.0`. A new image is created after then with the repository name inclusive. Then, push to docker hub: `sudo docker push jendyjasper/todo:v1.0`

<img width="930" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/dfdee96a-0af6-4914-9d46-ff6abcb8217e">

-----

Result on Dockerhub showing the pushed image

![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/b12950bd-1f51-4078-8030-861d5783280a)

-----

![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/b9c59ef1-38b0-4aa2-8ee3-1cfcba882455)


## Part 3
-  Write a Jenkinsfile that will simulate a Docker Build and a Docker Push to the registry
-  Connect your repo to Jenkins
-  Create a multi-branch pipeline
-  Simulate a CI pipeline from a feature and master branch using previously created Jenkinsfile
-  Ensure that the tagged images from your Jenkinsfile have a prefix that suggests which branch the image was pushed from. For example, feature-0.0.1.
-  Verify that the images pushed from the CI can be found at the registry.

**Below is my Jenkinsfile contents:**
```
pipeline {
    agent any

    environment { 
        //use the BRANCH_NAME and Build_ID environment variables to tag the image versions
        VERSION = "${env.BRANCH_NAME}-V1.${env.BUILD_ID}"
}

    stages {

        stage('Docker Build') {
            steps {
                sh 'sudo docker build -t todo:${VERSION} .'
                
            }
        }

        stage('Change tagname') {
            steps {
                sh 'sudo docker tag todo:${VERSION} jendyjasper/todo:${VERSION}'
            }        
        }

        stage ('Push to Docker Hub') {
            steps {
                sh 'sudo docker push jendyjasper/todo:${VERSION}'
            }
        }
        
        stage ('Delete Image Locally') {
            steps{
                sh 'sudo docker rmi jendyjasper/todo:${VERSION}'
                sh 'sudo docker rmi todo:${VERSION}'
            } 
        }
    }
}
```

-----

The jenkins CI dashboard showing both master branch and feature branch

<img width="1422" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/a5c99eb0-99d3-41db-9eb2-591d910ca05a">

-----

<img width="1422" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/c6f0493a-be82-4267-bda3-b33d8c4d398b">

-----

<img width="1422" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/a014a20a-bb04-455f-848f-d9632cb3578b">

-----

<img width="1422" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/a1628fb3-7059-4d1b-9815-12c0fb1aea7d">

-----
**The docker hub registry showing the images pushed to docker hub, with the branches showing on the tag name. You will see the images pushed from the feature branch have a prefix of feature while those pushed from the main branch have prefix of main**

<img width="1422" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/f2ed373d-c297-43e2-9012-b9044a710180">



-----

# Deployment with Docker Compose

1.  First, install Docker Compose on your workstation from [here](https://docs.docker.com/compose/install/)
2.  Create a file, name it tooling.yaml
3.  Begin to write the Docker Compose definitions with YAML syntax. The YAML file is used for defining services, networks, and volumes:

```
version: "3.9"
services:
  tooling_frontend:
    build: .
    ports:
      - "5000:80"
    volumes:
      - tooling_frontend:/var/www/html
```
  -  version: Is used to specify the version of Docker Compose API that the Docker Compose engine will connect to. This field is optional from docker compose version v1.27.0. You can verify your installed version with:
  -  service: A service definition contains a configuration that is applied to each container started for that service. In the snippet above, the
    only service listed there is tooling_frontend. So, every other field under the tooling_frontend service will execute some commands that relate only to that service. Therefore, all the below-listed fields relate to the tooling_frontend service.
  -  build: Requires it to build the image from the Dockerfile
  -  port: the port number of the host machine binded to the port number exposed on the docker container - hostPort:containerPort
  -  volumes: Saves the storage space of the container to the host machine for persistency  
  -  links: Creates a additional connection name that allows the running containers to communicate with each other using the specified link names

```
version: "3.9"
services:
  tooling_frontend:
    build: .
    ports:
      - "5000:80"
    volumes:
      - tooling_frontend:/var/www/html
    links:
      - db
  db:
    image: mysql:5.7
    restart: always
    environment:
      MYSQL_DATABASE: <The database name required by Tooling app >
      MYSQL_USER: <The user required by Tooling app >
      MYSQL_PASSWORD: <The password required by Tooling app >
      MYSQL_RANDOM_ROOT_PASSWORD: '1'
    volumes:
      - db:/var/lib/mysql
volumes:
  tooling_frontend:
  db:
```

-  Run the command to start the containers: `docker-compose -f tooling.yaml  up -d `
-  Verify that the compose is in the running status: `docker compose ls`

### Practice Task №2 – Complete Continous Integration With A Test Stage
  -  Document your understanding of all the fields specified in the Docker Compose file tooling.yaml
  -  Update your Jenkinsfile with a test stage before pushing the image to the registry.
  -  What you will be testing here is to ensure that the tooling site http endpoint is able to return status code 200. Any other code will be determined a stage failure.
  -  Implement a similar pipeline for the PHP-todo app.
  -  Ensure that both pipelines have a clean-up stage where all the images are deleted on the Jenkins server.

**Jenkinsfile**
```
pipeline {
    agent any

    environment { 
        //use the BRANCH_NAME and Build_ID environment variables to tag the image versions
        VERSION = "${env.BRANCH_NAME}-V1.${env.BUILD_ID}"
}

    stages {

        stage('Docker Compose UP') {
            steps { //use this to set the env var that is been used by todo.yml compose file
                sh 'sudo IMG_VERSION=${VERSION} docker compose -f todo.yaml up -d'
                
            }
        }

        stage('DB Migration') {
            steps {
                sh 'sleep 30'
                //run sudo docker exec -it todo php artisan migrate after successfull run by running it on the shell 
                sh 'sudo IMG_VERSION=${VERSION} docker compose -f todo.yaml exec -T  php_frontend php artisan migrate'
                
            }
        }

        stage ('Push to Docker Hub') {
            steps {
                //test if running container is reachable by checking if the status code is 200 and if it's reachable, 
                //push the image to docker hub and if not, print an error message
                sh '''#!/bin/bash
                    code=$(curl -s -o /dev/null -w "%{http_code}" 'http://34.204.0.208:5004/')
                    if [[ $code == "200" ]]; then
                        sudo docker push jendyjasper/todo:${VERSION}
                    else
                        echo "Website is unreachable, Please troubleshhot and fix the errors before pushing to docker hub. Deleting created images"
                    fi
                '''
            }
        }
        
        stage ('Docker Compose Down') { //can use docker-compuse too
            steps {
                sh 'sudo IMG_VERSION=${VERSION} docker compose -f todo.yaml down'
            }
        }

        stage ('Delete Image and Prune System') {
            steps{
                sh '''#!/bin/bash
                sudo docker rmi -f jendyjasper/todo:${VERSION}
                sudo docker rmi -f mysql:latest
                sudo docker system prune -f
                '''
            } 
        }

    }
}
```
-----
**Docker Compose File.yml**
```
version: "3.9"
services:
  php_frontend:
  #use double $$ to expand the jenkins env variable inside the container
    image: "jendyjasper/todo:$IMG_VERSION" #define the value of this on the terminal if using jenkins
    container_name: todo
    build: 
      context: .
      # tags: 
      #   - "jendyjasper/todo:1.1" # you can use this section to build images
      # for multiple repos or registries.
    ports:
      - "5000:80"
    volumes:
      - php_frontend:/var/www/html
    links:
      - db

  db:
    image: mysql:latest
    container_name: db
    restart: always
    environment:
      MYSQL_DATABASE: tododb
      MYSQL_USER: jendy
      MYSQL_PASSWORD: jendyjasper
      MYSQL_RANDOM_ROOT_PASSWORD: '1'
    volumes:
      - db:/var/lib/mysql
volumes:
  php_frontend:
  db:
```

-----

When the website doesn't return 200 ok http status code, this is the error on jenkins, and all the created images and containers stopped and deleted

<img width="1422" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/9f8623ad-9c8d-41ea-a33d-0bafa062ec1f">

-----

When the website returns 200 ok http status code, this is the confirmation of it pushing the image to docker hub registry

![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/9c9dd006-3e33-463a-8278-f056009ca2e1)

![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/ee7529f2-7967-485d-a459-a5bb2691a21e)

-----

Delete all the images and containers

![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/cc1362dd-9390-4b04-97b3-624269969f97)

-----

Image pushed to docker hub after succesfful test

![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/7433e9ec-9e6d-4b9d-a7f2-e6f2163d9d44)

