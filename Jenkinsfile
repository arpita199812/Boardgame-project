pipeline { 
    agent { label 'slave-1' }

    tools {
        maven 'maven3'
        jdk 'jdk17'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        DOCKER_HUB_CREDENTIALS = credentials('docker-hub-id')
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/arpita199812/Boardgame-project.git'
            }
        }

        stage('Clean') {
            steps {
                sh 'mvn clean'
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('Test') {
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

        stage('Package') {
            steps {
                sh 'mvn package'
            }
        }

        stage('Docker Login') {
            steps {
                script {
                   withCredentials([usernamePassword(credentialsId: 'docker-hub-key', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin https://index.docker.io/v1/'
                    }
                }
            }
        }

        stage('Set up Docker Buildx') {
            steps {
                script {
                    sh 'docker buildx --version || docker buildx create --use'
                }
            }
        }

        stage('Docker Build and Push') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-key') {
                    sh  'docker buildx build --platform linux/amd64,linux/arm64 -t arpita199812/boardgame-project:18 --push .'
                       
                    }
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                script {
                    sh 'docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image arpita199812/boardgame-project:18'
                }
            }
        }

        stage('OWASP ZAP Scan') {
            steps {
                script {
                    sh 'docker run --rm -v ${WORKSPACE}:/zap/wrk/:rw -t zaproxy/zap-stable zap-baseline.py -t http://3.87.190.86:8080 -r zap_report.html'
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    sh 'docker run -d -p 8080:8080 --name boardgame-container arpita199812/boardgame-project:18'
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: '**/target/*.jar', allowEmptyArchive: true
        }

        success {
            script {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        failure {
            echo 'Pipeline failed!'
        }
    }
}
