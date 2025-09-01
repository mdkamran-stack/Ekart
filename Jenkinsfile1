pipeline {
    agent any

    tools {
        jdk 'jdk11'
        maven 'maven3'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main',
                    changelog: false,
                    credentialsId: '86697b73-564b-4a1c-b35e-56ece7cb6e44',
                    poll: false,
                    url: 'https://github.com/mdkamran-stack/Ekart.git'
            }
        }

        stage('COMPILE') {
            steps {
                sh "mvn clean compile -DskipTests=true"
            }
        }

        stage('OWASP Scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('Sonarqube') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=Shopping-Cart \
                        -Dsonar.java.binaries=. \
                        -Dsonar.projectKey=Shopping-Cart'''
                }
            }
        }

        stage('Build') {
            steps {
                sh "mvn clean package -DskipTests=true"
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'docker') {
                        def app = docker.build("kamran112/your-image")
                        app.push()
                    }
                }
            }
        }
    }

    post {
        success {
            archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
            echo 'Build and deployment completed successfully.'
        }
        failure {
            echo 'Build failed. Check logs and scan reports.'
        }
    }
}
