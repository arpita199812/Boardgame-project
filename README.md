# Detailed CI/CD Pipeline Setup for a Java Project using Jenkins, Docker, SonarQube, Trivy, and AWS ECS

## Description

**Board Game Database Full-Stack Web Application.**
This web application displays lists of board games and their reviews. While anyone can view the board game lists and reviews, they are required to log in to add/ edit the board games and their reviews. The 'users' have the authority to add board games to the list and add reviews, and the 'managers' have the authority to edit/ delete the reviews on top of the authorities of users.  

## Technologies

- Java
- Spring Boot
- Amazon Web Services(AWS) EC2
- Thymeleaf
- Thymeleaf Fragments
- HTML5
- CSS
- JavaScript
- Spring MVC
- JDBC
- H2 Database Engine (In-memory)
- JUnit test framework
- Spring Security
- Twitter Bootstrap
- Maven

## Features

- Full-Stack Application
- UI components created with Thymeleaf and styled with Twitter Bootstrap
- Authentication and authorization using Spring Security
  - Authentication by allowing the users to authenticate with a username and password
  - Authorization by granting different permissions based on the roles (non-members, users, and managers)
- Different roles (non-members, users, and managers) with varying levels of permissions
  - Non-members only can see the boardgame lists and reviews
  - Users can add board games and write reviews
  - Managers can edit and delete the reviews
- Deployed the application on AWS EC2
- JUnit test framework for unit testing
- Spring MVC best practices to segregate views, controllers, and database packages
- JDBC for database connectivity and interaction
- CRUD (Create, Read, Update, Delete) operations for managing data in the database
- Schema.sql file to customize the schema and input initial data
- Thymeleaf Fragments to reduce redundancy of repeating HTML elements (head, footer, navigation)
  


## How to Run
Detailed CI/CD Pipeline Setup for a Java Project using Jenkins, Docker, SonarQube, Trivy, and AWS ECS

Overview
This guide covers the step-by-step process to set up a CI/CD pipeline for a Java application using Jenkins, Docker, SonarQube, Trivy, AWS ECS (Fargate), and other tools. The pipeline will automate the build, test, security scan, Docker image creation, and deployment of the Java application to AWS ECS.

### Prerequisites
- **AWS Account**: Access to EC2, ECS, ECR, IAM, and other related services.
- **Java Project**: Hosted in a GitHub repository.
- **Jenkins Installed**: On an EC2 instance or any server.
- **OWASP Dependency-Check**: On an EC2 agent server
- **Docker**: Installed on Jenkins server and agents.
- **SonarQube**: Installed via Docker on a dedicated port (9000).
- **Trivy**: Installed for security scanning.
- **GitHub Personal Access Token**: For repository access.
- **IAM Role for ECS**: To manage ECS deployment.

Step-by-Step Setup
 1. Launch EC2 Instances

1. **Log in to AWS Management Console**:
   - Go to the [AWS EC2 Console](https://console.aws.amazon.com/ec2/).

2. **Launch Two EC2 Instances**:
   - **Instance Type**: `t2.medium` with **15 GB storage** each.
   - **Instance 1 (Jenkins Master)**: Install Jenkins, Java, Docker, SonarQube.
   - **Instance 2 (Jenkins Agent)**: Install Java, Docker, Trivy, AWS CLI, Maven, OWASP Dependency-Check.

3. **Security Group Configuration**:
   - Allow inbound rules for **SSH (22)**, **HTTP (80)**, **Jenkins (8080)**, **SonarQube (9000)**, and other necessary ports.
   - Ensure both EC2 instances can communicate with each other.

#### 2. Install Required Software on EC2 Instances
##### **EC2 Instance 1: Jenkins Master Setup**

1. **Connect to Jenkins Master EC2 Instance**:
   ```bash
   ssh -i your-key.pem ubuntu@<Jenkins-Master-Public-IP>
   ```
2. **Update and Install Packages**:
   ```bash
   sudo yum update -y
   sudo yum install -y openjdk-17-jdk-headless docker git
   ```
3. **Install Jenkins**:
   ```bash
       sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
       https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
      echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
      https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
      /etc/apt/sources.list.d/jenkins.list > /dev/null
    sudo apt-get update
   sudo apt-get install jenkins
   ```

4. **Start Docker and Add Permissions**:
   ```bash
   sudo systemctl start docker
   sudo systemctl enable docker
   sudo usermod -aG docker ubuntu
   sudo usermod -aG docker Jenkins
   sudo chmod 666 /var/run/docker.sock
   ```

5. **Install AWS CLI**:
   ```bash
    sudo apt-get install unzip
   curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
   unzip awscliv2.zip
   sudo ./aws/install
   ```
6. **Install SonarQube Using Docker**:
   - Run the SonarQube Docker image:
   ```bash
   docker run -d -p 9000:9000 sonarqube:lts-community
   ```

7. **Open Jenkins Web Interface**:
   - Access Jenkins via `http://<Jenkins-Master-Public-IP>:8080`.
   - Unlock Jenkins using the initial admin password:
   ```bash
   sudo cat /var/lib/jenkins/secrets/initialAdminPassword
   ```

8. **Install Suggested Plugins in Jenkins**:
   - Install recommended plugins during the initial setup.

##### **EC2 Instance 2: Jenkins Agent Setup**

1. **Connect to Jenkins Agent EC2 Instance**:
   ```bash
   ssh -i your-key.pem ubuntu@<Jenkins-Agent-Public-IP>
   ```
2. **Update and Install Packages**:
   ```bash
   sudo yum update -y
   sudo yum install -y openjdk-17-jdk-headless docker maven git
   ```

3. **Install OWASP Dependency-Check and Trivy**:
   - **OWASP Dependency-Check**:
Go to Manage Jenkins -> Plugin -> Installed Plugin -> Dependency Check Plugin , then go to Tool -> Dependency-Check -> Name (DP ) -> Installed version (6.5.3)
   - **Trivy**:
   ```bash
         sudo apt-get install wget apt-transport-https gnupg lsb-release
         wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
         echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a 
         /etc/apt/sources.list.d/trivy.list
         sudo apt-get update
         sudo apt-get install trivy
         trivy --version
   ```

4. **Start Docker and Add Permissions**:
   ```bash
   sudo systemctl start docker
   sudo systemctl enable docker
   sudo usermod -aG docker ubuntu
   sudo adduser jenkinsuser
   sudo usermod -aG docker jenkinsuser
  chmod 777 /var/run/docker. sock
   ```

#### 3. Configure Jenkins Master to Use Jenkins Agent

1. **Add Jenkins Agent**:
   - Go to **Manage Jenkins** > **Manage Nodes and Clouds** > **New Node**.
   - Enter `Node Name` (e.g., `Slave-1`), select `Permanent Agent`, and configure:
     - **Remote Root Directory**: `/home/ubuntu/slave`
     - **Number of Executors**: `2`
     - **Launch Method**: SSH with credentials. (Private-key)

2. **Add SSH Credentials in Jenkins**:
   - Use EC2 key pair credentials for SSH access.

3. **Save and Launch Agent**.

#### 4. Install Required Jenkins Plugins
1. **Go to `Manage Jenkins` > `Manage Plugins`**:
   - Install the following plugins:
     - AWS Credentials
     - Docker Pipeline Plugin
     - Docker Step API
     - SonarQube Scanner Plugin
     - OWASP Dependency-Check Plugin
     - Trivy Plugin (if available)
     - Credentials Binding Plugin

2. **Configure Jenkins Plugins**:
   - **AWS Credentials**: Add AWS credentials (`Manage Jenkins` > `Manage Credentials` > `Add Credentials`).
   - **SonarQube**: 
     - Go to `Manage Jenkins` > `Configure System`.
     - Under `SonarQube servers`, add SonarQube server details.
   - **Docker Credentials**:
     - Store Docker Hub credentials (`Manage Jenkins` > `Manage Credentials` > `Add Credentials`).

#### 5. Set Up SonarQube for Code Quality Analysis
1. **Access SonarQube**:
   - Navigate to `http://<Jenkins-Master-Public-IP>:9000`.
   - Log in using default credentials (`admin/admin`).

2. **Configure SonarQube Project**:
   - Create a new project and generate a token.
   - Save the token for Jenkins configuration.

3. **Integrate SonarQube with Jenkins**:
   - Go to `Manage Jenkins` > `Configure System` > `SonarQube servers`.
   - Add SonarQube instance details (URL: `http://<Jenkins-Master-Public-IP>:9000`).
   - Add the SonarQube token under Jenkins credentials.

#### 6. Configure Docker Credentials in Jenkins
1. **Add Docker Hub Credentials**:
   - Go to **Manage Jenkins** > **Manage Credentials** > **Global** > **Add Credentials**.
   - Select **Username with password**.
   - Enter your Docker Hub **username** and **personal access token** as **password**.
   - Set **ID** as `docker-hub-credentials`.

2. **Set Up AWS Credentials**:
   - Add AWS credentials (Access Key ID and Secret Access Key) under **Global Credentials** with **ID** as `AWS-Credential-ID`.

#### 7. Set Up Webhook in GitHub Repository
1. **Navigate to your GitHub repository**.
2. Go to **Settings** > **Webhooks** > **Add Webhook**.
3. Set the **Payload URL** to your Jenkins URL with the `/github-webhook/` path:
   - Example: ` Jenkins_URL/generic-webhook-trigger/invoke?token=github_token’
4. Set **Content type** to `application/json`.
5. Choose **Just the push event** option.
6. Click **Add Webhook**.

#### 8. Create and Configure AWS Resources
1. **Create an ECR Repository**:
   - Go to the AWS Management Console.
   - Navigate to **ECR** > **Repositories** > **Create repository**.
   - Name your repository (e.g., `broadgame-project `).
   - Make sure the repository is created in the same region where your ECS cluster will be deployed.

2. **Create an ECS Cluster**:
   - Go to **ECS** in the AWS Management Console.
   - Create a new cluster using the **Fargate** launch type.
   - Name the cluster (e.g., ` Broadgame-proj-cluster `).

3. **Create Task Definition**:
   - Define the task with the necessary configurations (container definitions, resource limits).

4. **Create an ECS Service**:


   - Create a service linked to the task definition and cluster created.
   - Choose a deployment type (e.g., **Rolling update**).

5. **IAM Role for ECS**:
   - Create an IAM role for ECS with the following policies:
     - `AmazonECSTaskExecutionRolePolicy`
     - `AmazonECS_FullAccess`
     - Name the role (e.g., `MyECSTaskRole`).

#### 9. Create Jenkins Pipeline Script
1. **Go to Your Jenkins Dashboard**:
   - Click **New Item** > **Pipeline**.
   - Enter a name for your pipeline (e.g., `Broadgame-Pipeline `).
   - Choose **Pipeline** and click **OK**.

2. **Configure Pipeline Script**:
   - Scroll down to the **Pipeline** section.
   - Choose **Pipeline script from SCM** and paste the github repository link with branch name:

3. **Save the Pipeline** and click **Build Now** to start the pipeline.

#### 10. Verify the Deployment
1. **Monitor Jenkins Console Output**:
   - Ensure each stage completes successfully without errors.

2. **Validate ECR Push**:
   - Go to AWS ECR and check if the Docker image is pushed.

3. **Validate ECS Deployment**:
   - Go to ECS, select the cluster, and verify the service and tasks are running.

4. **Test Application**:
   - Access the deployed application via the ECS service endpoint.

### Conclusion

This guide covers the detailed steps to set up a Jenkins-based CI/CD pipeline integrated with Docker, SonarQube, Trivy, and AWS ECS. It ensures automated code quality checks, security scans, Docker image creation, and deployment.

### Additional Resources

- [Jenkins Documentation](https://www.jenkins.io/doc/)
- [AWS ECS Documentation](https://docs.aws.amazon.com/ecs/)
- [SonarQube Documentation](https://docs.sonarqube.org/)
- [Trivy Documentation](https://aquasecurity.github.io/trivy/)
- [Docker Documentation](https://docs.docker.com/)

4. To use initial user data, use the following credentials.
  - username: bugs    |     password: bunny (user role)
  - username: daffy   |     password: duck  (manager role)

![Screenshot 2024-08-28 213244](https://github.com/user-attachments/assets/560cd6e7-2aee-45d7-9d51-95b44a5845c9)

![Screenshot 2024-08-28 212941](https://github.com/user-attachments/assets/40f79f45-e489-434d-81db-543603433b86)

![Screenshot 2024-08-28 213334](https://github.com/user-attachments/assets/ce9ecc80-c11c-4034-9dd5-522d489599c4)

![Screenshot 2024-08-28 213437](https://github.com/user-attachments/assets/28c07e51-e715-40f9-8809-17f8ab443068)

![Screenshot 2024-08-28 213448](https://github.com/user-attachments/assets/48801221-4db1-4324-9c51-dc74197ff7db)

5. You can also sign-up as a new user and customize your role to play with the application! 😊
