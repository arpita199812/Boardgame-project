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
                    dependencyCheck additionalArguments: '--failOnCVSS 7 --format XML --format HTML', 
                                    odcInstallation: 'Dependency-Check 6.5.3', 
                                    out: 'dependency-check-report'
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

    post {
        always {
            // Publish Dependency-Check reports
            dependencyCheckPublisher pattern: 'dependency-check-report/dependency-check-report.xml'
        }
    }
}
