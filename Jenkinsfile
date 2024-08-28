pipeline { 
    agent { label 'slave-1' }

    tools {
        maven 'maven3'
        jdk 'jdk17'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
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
                dependencyCheck additionalArguments: '--scan ./ --format HTML --nvdApiKey nvd-api-key', odcInstallation: 'DP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
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
                    withDockerRegistry(credentialsId: 'docker-hub-id', toolName: 'docker') {
                        sh 'docker build -t image1 .'
                        sh 'docker tag image1 arpita199812/boardgame-project:latest'
                        sh 'docker push arpita199812/boardgame-project:latest'
                    }
                }
            }
        }

        stage('TRIVY') {
            steps {
                sh 'trivy image arpita199812/boardgame-project:latest'
            }
        }
    }
}
