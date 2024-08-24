pipeline { 
    agent  { label 'slave-1'}

    tools {
        maven 'maven3'
        jdk 'jdk17'
    }

    environment {
        // Specify the SonarQube environment name from Jenkins global configuration
        SCANNER_HOME= tool 'sonar-scanner'
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
                    // Run SonarQube analysis
                    withSonarQubeEnv('sonar-server') {
                        sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Boardgame-project -Dsonar.projectKey=Broadgame-project \
                        -Dsonar.java.binaries=target'''

                        sh "echo $SCANNER_HOME"
                    }
                }
            }
        }

        stage('Package') {
            steps {
                sh 'mvn package'
            }
        }

        stage('Install') {
            steps {
                sh 'mvn install'
            }
        }
    }

    post {
        always {
            // Optional: Archive the SonarQube analysis report or other artifacts
            archiveArtifacts artifacts: '**/target/*.jar', allowEmptyArchive: true
        }

        success {
            script {
                // Wait for the SonarQube analysis report to be processed on the server
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        failure {
            // Send notifications or take other actions on failure
            echo 'Pipeline failed!'
        }
    }
}
