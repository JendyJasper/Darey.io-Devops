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
