pipeline { 
    agent any

    tools {
        maven 'maven3'
        jdk 'jdk17'
    }

    environment {
        // Specify the SonarQube environment name from Jenkins global configuration
        SONARQUBE_ENV = 'SonarQube'
    }

    stages {
        stage('Git-Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/arpita199812/Boardgame-project.git'
            }
        }

        stage('Clean') {
            steps {
                bat 'mvn clean'
            }
        }

        stage('Compile') {
            steps {
                bat 'mvn compile'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    // Run SonarQube analysis
                    withSonarQubeEnv(SONARQUBE_ENV) {
                        bat 'mvn sonar:sonar -Dsonar.projectKey=Boardgame-project'
                    }
                }
            }
        }

        stage('Test') {
            steps {
                bat 'mvn test'
            }
        }

        stage('Package') {
            steps {
                bat 'mvn package'
            }
        }

        stage('Install') {
            steps {
                bat 'mvn install'
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
