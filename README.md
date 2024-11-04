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
_______________________________________________________________________________________________________________________________________________
1. Open tak/CoreConfig.example.xml and set a database password
2. Make any other configuration changes you need
TAK Server Database Container Setup:
    1. Build TAK server database image:
    > docker build -t takserver-db:"$(cat tak/version.txt)" -f docker/Dockerfile.takserver-db .
    2. Create a new docker network for the current tak version:
    > docker network create takserver-"$(cat tak/version.txt)"
3. The TAK Server database container can be configured to persist data directly to the host or only
within the container.
a. To persist to the host, create an empty host directory (unless you have a directory from a
previous docker install you want to reuse). For upgrading purposes, we recommend that you
keep the takserver database directory outside of the 'takserver-docker-<version>' directory
structure.
> docker run -d -v <absolute path to takserver database directory>:/var/lib/postgresql/data:z -v
$(pwd)/tak:/opt/tak:z -it -p 5432:5432 --network takserver-"$(cat tak/version.txt)" --network-alias tak-
database --name takserver-db-"$(cat tak/version.txt)" takserver-db:"$(cat tak/version.txt)"
b. To run TAK server database with container only persistence
> docker run -d -v $(pwd)/tak:/opt/tak:z -it -p 5432:5432 --network takserver-"$(cat tak/version.txt)" --
network-alias tak-database --name takserver-db-"$(cat tak/version.txt)" takserver-db:"$(cat
tak/version.txt)"
37
TAK Server Container Setup:
1. Build TAK Server image:
> docker build -t takserver:"$(cat tak/version.txt)" -f docker/Dockerfile.takserver .
2. Running TAK Server container: use -p <host port>:<container port> to map any additional ports you
have configured. Adding new inputs or changing ports while the container is running will require
the container to be recreated so that the new port mapping can be added.
> docker run -d -v $(pwd)/tak:/opt/tak:z -it -p 8089:8089 -p 8443:8443 -p 8444:8444 -p 8446:8446 -p
9000:9000 -p 9001:9001 --network takserver-"$(cat tak/version.txt)" --name takserver-"$(cat
tak/version.txt)" takserver:"$(cat tak/version.txt)"
2. Before using the TAK Server, you must setup the certificates for secure operation. If you have
already configured certificates you can skip this step. You can also copy existing certificates into
'tak/certs/files' and a UserAuthetication.xml file into 'tak/' to reuse existing certificate authentication
settings. Any change to certificates while the container is running will require either a TAK server
restart or container restart. Additional certificate details can be found in Appendix B.
a. Edit tak/certs/cert-metadata.sh
b. Generate root ca
> docker exec -it takserver-"$(cat tak/version.txt)" bash -c "cd /opt/tak/certs && ./makeRootCa.sh"
c. Generate server cert
> docker exec -it takserver-"$(cat tak/version.txt)" bash -c "cd /opt/tak/certs && ./makeCert.sh server
takserver"
d. Create client cert(s)
> docker exec -it takserver-"$(cat tak/version.txt)" bash -c "cd /opt/tak/certs && ./makeCert.sh client
<user>"
e. Restart takserver to load new certificates
> docker exec -d takserver-"$(cat tak/version.txt)" bash -c "cd /opt/tak/ && ./configureInDocker.sh"
f. Tail takserver logs from the host. Once TAK server has successfully started, proceed to the
next step.
> tail -f tak/logs/takserver-messaging.log
> tail -f tak/logs/takserver-api.log
38
3. Accessing takserver
Create admin client certificate for access on secure port 8443 (https):
> docker exec takserver-"$(cat tak/version.txt)" bash -c "cd /opt/tak/ && java -jar
utils/UserManager.jar certmod -A certs/files/<client cert>.pem"
Hardened TAK Server Setup:
The hardened TAK Database and Server containers provide additional security by including the use of
secure Iron Bank base images, container health checks, and minimizing user privileges within the
containers.
The hardened TAK images are available in a zip file, takserver-docker-hardened-<version>.zip. The
steps for setting up the hardened containers are similar to the standard docker installation steps given
above except for the following:
Certificate Generation:
The certificate generation container is only required to run once for TAK Server initialization. Run all
commands in this section from the root of the unzipped hardened docker contents.
1. Build the Certificate Authority Setup Image:
< docker build -t ca-setup-hardened --build-arg ARG_CA_NAME=<CA_NAME> --build-arg
ARG_STATE=<ST> --build-arg ARG_CITY=<CITY> --build-arg
ARG_ORGANIZATIONAL_UNIT=<UNIT> -f docker/Dockerfile.ca .
2. Run the Certificate Authority Setup Container: If certificates have previously been generated and
exist in the tak/cert/files path when building the ca-setup-hardened image then certificate generation
will be skipped at runtime.
<docker run --name ca-setup-hardened -it -d ca-setup-hardened
3. Copy the generated certificates for TAK Server:
> docker cp ca-setup-hardened:/tak/certs/files files
> [ -d tak/certs/files ] || mkdir tak/certs/files \
&& docker cp ca-setup-hardened:/tak/certs/files/takserver.jks tak/certs/files/ \
&& docker cp ca-setup-hardened:/tak/certs/files/truststore-root.jks tak/certs/files/ \
&& docker cp ca-setup-hardened:/tak/certs/files/fed-truststore.jks tak/certs/files/ \
&& docker cp ca-setup-hardened:/tak/certs/files/admin.pem tak/certs/files/ \
&& docker cp ca-setup-hardened:/tak/certs/files/config-takserver.cfg tak/certs/files/
39
TAK Server Database Hardened Container Setup:
1. Building the hardened docker images requires creating an Iron Bank/Repo1 account to access the
approved base images. To create an account, follow the instructions in the IronBank Getting Started
page. To download the base images via the CLI, see the instructions in the Registry Access section.
After obtaining the necessary credentials, run:
< docker login registry1.dso.mil
2. Follow the instructions in the TAK Server CoreConfig Setup section and update the <connection-
url> tag with the hardened TAK Database container name. For example:
<connection url="jdbc:postgresql://tak-database-hardened-<version>:5432/cot" username="martiuser"
password=<password>/>
3. Create a new docker network for the current tak version:
> docker network create takserver-net-hardened-"$(cat tak/version.txt)"
Ensure in the db-utils/pg_hba.conf file that there is an entry for the subnet of the hardened takserver
network. To determine the subnet of the network:
< docker network inspect takserver-net-hardened-"$(cat tak/version.txt)"
Or to specify the subnet on network creation:
< docker network create takserver-net-hardened-"$(cat tak/version.txt)" --subnet=<subnet>
4. Build the hardened TAK Database image:
<docker build -t tak-database-hardened:"$(cat tak/version.txt)" -f docker/Dockerfile.hardened-
takserver-db .
5. Run the hardened TAK Database container:
< docker run --name tak-database-hardened-"$(cat tak/version.txt)" --network takserver-net-
hardened-"$(cat tak/version.txt)" --network-alias tak-database -d tak-database-hardened:"$(cat
tak/version.txt)" -p 5432:5432
TAK Server Hardened Container Setup
1. Build the hardened TAK Server image:
< docker build -t takserver-hardened:"$(cat tak/version.txt)" -f docker/Dockerfile.hardened-takserver .
2. Run the hardened TAK Server container:
40
< docker run --name takserver-hardened-"$(cat tak/version.txt)" --network takserver-net-hardened-"$
(cat tak/version.txt)" -p 8089:8089 -p 8443:8443 -p 8444:8444 -p 8446:8446 -t -d takserver-
hardened:"$(cat tak/version.txt)"
Configuring Certificates
1. Get the admin certificate fingerprint
> docker exec -it ca-setup-hardened bash -c "openssl x509 -noout -fingerprint -md5 -inform pem -in
files/admin.pem | grep -oP 'MD5 Fingerprint=\K.*'"
2. Add the certificate fingerprint as the admin after the hardened TAK server container has started
(about 60 seconds)
> docker exec -it takserver-hardened-"$(cat tak/version.txt)" bash -c 'java -jar
/opt/tak/utils/UserManager.jar usermod -A -f <admin cert fingerprint> admin'
Useful Commands
*To run these commands on the hardened containers, add the -hardened suffix to the container names.
• View images:
> docker images takserver
> docker images takserver-db
• View containers
All: > docker ps -a
Running: > docker ps
Stopped: > docker ps -a | grep Exit
• Exec into container
> docker exec -it takserver-"$(cat tak/version.txt)" bash
> docker exec -it takserver-db-"$(cat tak/version.txt)" bash
• Exec command in container
> docker exec -it takserver-"$(cat tak/version.txt)" bash -c "<command>"
> docker exec -it takserver-db-"$(cat tak/version.txt)" bash -c "<command>"
• Tail takserver logs
> tail -f tak/logs/takserver-messaging.log
> tail -f tak/logs/takserver-api.log
• Restart TAK server
41
> docker exec -d takserver-"$(cat tak/version.txt)" bash -c "cd /opt/tak/ &&
./configureInDocker.sh"
• Start/Stop container:
> docker <start/stop> takserver-"$(cat tak/version.txt)"
> docker <start/stop> takserver-db-"$(cat tak/version.txt)"
• Remove container:
> docker rm -f takserver-"$(cat tak/version.txt)"
> docker rm -f takserver-db-"$(cat tak/version.txt)"
_______________________________________________________________________________________________________________________________________________
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
