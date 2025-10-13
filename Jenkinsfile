pipeline {
    agent none
    stages {
        stage('Checkout') {
            agent { docker { image 'php:8.2-cli' } }
            steps {
                sh 'php --version'
            }
        }
        stage('Compilation') {
            agent { docker { image 'php:8.2-cli' } }
            steps {
                sh 'echo "Compilando..."'
            }
        }
        stage('Build') {
            agent { docker { image 'php:8.2-cli' } }
            steps {
                sh 'echo "docker build -t my-php-app ."'
            }
        }
        stage('Deploy') {
            agent { docker { image 'php:8.2-cli' } }
            steps {
                sh 'echo "docker run my-php-app ."'
            }
        }
        stage('SonarQube Scan') {
            agent { docker { image 'sonarsource/sonar-scanner-cli:latest' }
            }
            environment {
                SONAR_TOKEN = credentials('sonarqube-token')
            }
            steps {
                sh '''
                    sonar-scanner \
                      -Dsonar.projectKey=dvwa-sast-sonarqube \
                      -Dsonar.sources=. \
                      -Dsonar.host.url=http://host.docker.internal:9000 \
                      -Dsonar.login=$SONAR_TOKEN \
		      -Dsonar.report.export.path=sonar-report.json
                '''
            }
        }
        post {
            always {
               archiveArtifacts artifacts: 'sonar-report.json', fingerprint: true
            }
        }

    }
}
