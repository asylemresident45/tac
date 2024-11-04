Project Title: Cloud-based Application Deployment and Management
Intent

The objective of this project is to deploy multiple containers using Docker and Docker Compose. This project aims to enhance practical skills in docker while enhancing your ability to manage infrastructure, application deployment, and containerization.
Requirements

    Deploy a Docker-Compose file on your work laptop or vm that has multiple containerized services, including but not limited to:
        A WordPress/Moodle/Piwigo instance
        A proxy server (NGINX or Apache)
        A MySQL or MariaDB database
        A Tak server using the MySQL or Maria DB for the database portion
        Bonus: Implement a load balancer to handle traffic.
    Note: Configuration of the WordPress site itself is not required once it is deployed. It only needs to be accessible via IP address.

Tasks

    Install Docker and Docker Compose on the virtual machine.
    Use Docker images to set up WordPressMoodle/Piwingo, NGINX/Apache, MySQL/MariaDB, and the Tak server.
    Define the services in a docker-compose.yml file.
    (Optional) Implement a load balancer with assigned port numbers.

Overall Requirements

    Docker/Docker Compose: Utilize Docker for containerizing the applications, and Docker Compose for managing multi-container setups.

Recommended Workflow

    Local Testing: build and test containers locally using Docker/Docker Compose.
    Code Management: Maintain all code and configurations on GitHub. Share code with each other and update as you go.

Collaboration

Students will work in teams, with each student working independently on their own project. However, they are encouraged to collaborate with one another and share resources. The goal is for each student to have hands-on practice with deploying containerized applications.
General Notes for Students

    Focus on Understanding: Before diving into solutions, take time to understand the core intent of each project.
    Collaborate: Use the strength of your team. Discuss, brainstorm, and allocate tasks based on individual strengths.
    Creative Freedom: There's no single 'right' solution. Choose the tools and methods you believe best fit the project's intent, keeping Docker and Docker Compose as central tools.
    Documentation: As you work, document your decisions, challenges, and solutions. This will be invaluable for reflection and presentation.
