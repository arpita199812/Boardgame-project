pipeline {
    agent { label 'slave-1' }

    tools {
        maven 'maven3'
        jdk 'jdk17'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        DOCKER_CREDENTIALS_ID = 'docker-hub-credentials'
        AWS_REGION = 'us-east-1' 
        ECR_REPOSITORY = 'boardgame-project' 
        ECS_CLUSTER = 'Boardgame-proj-cluster' 
        ECS_SERVICE = 'broadgame-service' 
        AWS_CREDENTIALS_ID = 'AWS_CREDENTIALS_ID' 
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/arpita199812/Boardgame-project.git'
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('Test Cases') {
            steps {
                sh 'mvn test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv('sonar-server') {
                        sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Boardgame-project \
                            -Dsonar.projectKey=Boardgame-project -Dsonar.java.binaries=target'''
                    }
                }
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                script {
                    env.DEPENDENCYCHECK_APIKEY = '6d6f8a54-3927-4686-96ad-e7cd1eb26044'
                    dependencyCheck additionalArguments: '--scan ./ --format HTML ', odcInstallation: 'DP'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.html'
                }
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Docker Build & Push to ECR') {
            steps {
                script {
                    withAWS(credentials: "${AWS_CREDENTIALS_ID}", region: "${AWS_REGION}") {
                        sh '''
                            # Log in to Amazon ECR
                            $(aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin 730335449419.dkr.ecr.us-east-1.amazonaws.com)
                            
                            # Build Docker image
                            docker build -t boardgame-project .
                            
                            # Tag Docker image
                            docker tag boardgame-project:latest 730335449419.dkr.ecr.us-east-1.amazonaws.com/boardgame-project:latest
                            
                            # Push Docker image to ECR
                            docker push 730335449419.dkr.ecr.us-east-1.amazonaws.com/boardgame-project:latest
                        '''
                    }
                }
            }
        }

        stage('Deploy to ECS') {
            steps {
                script {
                    withAWS(credentials: "${AWS_CREDENTIALS_ID}", region: "${AWS_REGION}") {
                        sh '''
                            # Update ECS service with the new image
                            aws ecs update-service --cluster $ECS_CLUSTER --service $ECS_SERVICE \
                            --force-new-deployment --region $AWS_REGION
                        '''
                    }
                }
            }
        }

        stage('TRIVY') {
            steps {
                sh 'trivy image 730335449419.dkr.ecr.us-east-1.amazonaws.com/boardgame-project:latest'
            }
        }
    }
}
