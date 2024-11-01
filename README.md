Lab 6: Creating a TAC Container Using Docker

Objective

This lab will guide you through the process of creating a TAC container using Docker. You will learn to set up a Docker container running the Tactical Air Control (TAC) application.

Prerequisites

    Docker installed on your machine.
    The takserver-docker-5.2-RELEASE-43.zip file downloaded and accessible.

Pre-lab Requirements

    Download Google Authenticator:

    Download the Google Authenticator app on your phone.

    Access TAK Website:

    Go to https://tak.gov.

    Create an Account:

    Click Create Account in the top right corner.

    Input Email:

    Input your email and click OK at the popup.

    Verify Email:

    Go to your email and verify your email address.

    Select User Type:

    Ensure Public User is selected and click Next.

    Input Personal Details:

    Input your personal details and click Submit.

    Login to TAK Account:

    Log in to your TAK account.

    Navigate to TAK Server:

    At the top of the screen, go to Products > Other > TAK Server.

    Download TAK Server Files:

    Download takserver-docker-5.2-release-43.zip or any future versions.

Lab Overview

In this lab, you will:

    Create a Docker container for the TAC application.
    Build and run the TAC server using Docker.

Lab Instructions

    Setting Up Project Directory:

    Start by making a new directory using the mkdir command:

    bash

mkdir tak-docker
cd tak-docker

Place the takserver-docker-5.2-RELEASE-43.zip file inside the directory you just created.

Unzip the Downloaded File:

Next, run the following command to extract the contents of the ZIP file:

bash

unzip takserver-docker-5.2-RELEASE-43.zip

This command unzips the takserver-docker-5.2-RELEASE-43.zip file, allowing you to access the necessary files for building and running the TAC container. The extracted contents include Dockerfiles and configurations needed for the next steps.

Navigate to the Unzipped Directory:

Now, change your current working directory to the one just extracted:

bash

cd takserver-docker-5.2-RELEASE-43/

This command ensures you are in the correct directory to access the Dockerfiles and other resources required for the subsequent commands.

Build the Database Docker Image:

Next, you will build the database Docker image by executing the following command:

bash

docker build -t takserver-db:"$(cat tak/version.txt)" -f docker/Dockerfile.takserver-db .

This command builds a Docker image named takserver-db and tags it with the version specified in version.txt. The -f option indicates the specific Dockerfile to use. Building this image is essential for setting up the database component of your TAC server.

Create a Docker Network:

Now, create a Docker network for your containers by running:

bash

docker network create takserver-"$(cat tak/version.txt)"

This command establishes a new Docker network named takserver-<version>. A dedicated network allows the containers to communicate securely while isolating them from others, maintaining the integrity of your application.

Run the Database Container:

Next, start the database container with the following command:

bash

docker run -d -v $(pwd)/tak:/opt/tak:z -p 5432:5432 --network takserver-"$(cat tak/version.txt)" --name takserver-db takserver-db:"$(cat tak/version.txt)"

This command runs the takserver-db image in detached mode. The -v option mounts the tak directory to /opt/tak in the container for data persistence. The -p option maps port 5432 on the host to port 5432 in the container, enabling access to the database.

Build the TAC Server Docker Image:

Now, proceed to build the TAC server Docker image by executing:

bash

docker build -t takserver:"$(cat tak/version.txt)" -f docker/Dockerfile.takserver .

This command builds another Docker image named takserver using the Dockerfile.takserver. This image contains the main application for TAC.

Run the TAC Server Container:

Next, run the TAC server container with the following command:

bash

docker run -d -v $(pwd)/tak:/opt/tak:z -p 8089:8089 -p 8443:8443 -p 8444:8444 -p 9000:9000 --network takserver-"$(cat tak/version.txt)" --name takserver takserver:"$(cat tak/version.txt)"

This command starts the takserver image in detached mode, with multiple ports (8089, 8443, 8444, 9000) mapped from the host to the container. This setup allows external access to different services provided by the TAC application.

List Running Containers:

To verify that your containers are running correctly, use the following command:

bash

docker ps

This command lists all currently active Docker containers, allowing you to check the status of your TAC server and database.

Stop a Running Container:

If you need to stop a running container, you can do so with:

bash

docker stop <container_name_or_id>

Replace <container_name_or_id> with the actual name or ID of the container. This command stops the specified container, necessary when you need to make changes, troubleshoot, or gracefully shut down services.

Remove a Stopped Container:

Finally, to clean up your environment, you can remove a stopped container by running:

bash

    docker rm <container_name_or_id>

    Again, replace <container_name_or_id> with the appropriate name or ID. This command deletes the specified container, helping you manage system resources and avoid clutter in your Docker environment.

Conclusion

By following these steps, you have successfully created a TAC container using Docker. You’ve learned how to structure your project, build Docker images, and run containers effectively. This knowledge is essential for managing applications in containerized environments.
