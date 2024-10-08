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
        ECS_CLUSTER = 'Broadgame-proj-cluster' 
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
                    env.DEPENDENCYCHECK_APIKEY = 'my-nvd-api-key'
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
                         # Get ECR login password and log in to Amazon ECR
                         aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin <aws-account-id>.dkr.ecr.us-east-1.amazonaws.com
                    
                        # Verify login
                        docker info
                    
                        # Build Docker image
                        docker build -t boardgame-project .
                    
                        # Tag Docker image
                        docker tag boardgame-project:latest <aws-account-id>.dkr.ecr.us-east-1.amazonaws.com/boardgame-project:latest
                    
                        # Push Docker image to ECR
                        docker push <aws-account-id>.dkr.ecr.us-east-1.amazonaws.com/boardgame-project:latest
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
                            # Check ECS cluster existence
                            aws ecs describe-clusters --clusters ${ECS_CLUSTER} --region ${AWS_REGION}

                            # Update ECS service with the new image
                            aws ecs update-service --cluster ${ECS_CLUSTER} --service ${ECS_SERVICE} \
                            --force-new-deployment --region ${AWS_REGION}
                        '''
                    }
                }
            }
        }

        stage('TRIVY') {
            steps {
                sh 'trivy image <aws-account-id>.dkr.ecr.us-east-1.amazonaws.com/boardgame-project:latest'
            }
        }
    }
}
