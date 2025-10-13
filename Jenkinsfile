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
            agent {
                docker { image 'sonarsource/sonar-scanner-cli:latest' }
            }
            steps {
                script {
                    // Ejecuta el análisis de SonarQube dentro del entorno configurado
                    withSonarQubeEnv('SonarQube') {
                        sh '''
                            echo "Ejecutando análisis con SonarQube..."
                            sonar-scanner \
                              -Dsonar.projectKey=dvwa-sast-sonarqube \
                              -Dsonar.sources=. \
                              -Dsonar.login=$SONAR_AUTH_TOKEN \
                              -Dsonar.report.export.path=sonar-report.json
                        '''
                    }
                }
            }
            post {
                always {
                    // Archiva el reporte JSON como artefacto en Jenkins
                    archiveArtifacts artifacts: 'sonar-report.json', fingerprint: true
                    echo "Reporte de SonarQube archivado correctamente."
                }
            }
        }

        stage('Quality Gate') {
            steps {
                // Espera el resultado de la Quality Gate (puede bloquear el pipeline si falla)
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline finalizado con éxito.'
        }
        failure {
            echo 'Pipeline falló. Revisa el reporte en Jenkins y SonarQube.'
        }
        always {
            echo 'Pipeline ejecutado completamente (post actions finalizadas).'
        }
    }
}

