pipeline {
    agent { label 'slave-1' }

    tools {
        maven 'maven3'
        jdk 'jdk17'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        DOCKER_CREDENTIALS_ID = 'docker-hub-credentials'
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
                     dependencyCheck additionalArguments: '--scan ./ --format HTML ', odcInstallation: 'DP',
                     dependencyCheckPublisher pattern: '/home/ubuntu/slave/workspace/pipeline/dependency-check-report.html'
                }
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS_ID) {
                        sh 'docker build -t arpita199812/boardgame-project .'
                        sh 'docker push arpita199812/boardgame-project:latest'
                    }
                }
            }
        }

        stage('TRIVY') {
            steps {
                sh 'docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest image --scanners vuln arpita199812/boardgame-project:latest'
            }
        }
    }
}
