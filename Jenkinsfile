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
        stage('Git-Checkout') {
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
                        sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Boardgame-project -Dsonar.projectKey=Broadgame-project \
                        -Dsonar.java.binaries=target'''
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
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-id', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    }
                }
            }
        }

        stage('Docker Build and Push') {
            steps {
                script {
                    docker.withRegistry('', 'docker-hub-id') {
                        def app = docker.build("arpita199812/boardgame-project:${env.BUILD_NUMBER}")
                        app.push()
                    }
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                script {
                    sh 'docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image arpita199812/boardgame-project:${env.BUILD_NUMBER}'
                }
            }
        }

        stage('OWASP ZAP Scan') {
            steps {
                script {
                    sh 'docker run --rm -v $(pwd):/zap/wrk/:rw -t  zaproxy/zap-stable zap-baseline.py -t http://3.87.190.86:8080 -r zap_report.html'
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    // Run the Docker container
                    sh 'docker run -d -p 8080:8080 --name boardgame-container arpita1999812/boardgame-project:${env.BUILD_NUMBER}'
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
