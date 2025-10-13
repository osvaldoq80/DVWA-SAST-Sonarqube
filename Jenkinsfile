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
                docker {
                    image 'sonarsource/sonar-scanner-cli:latest'
                    args '--entrypoint=""'  // evita el error del ENTRYPOINT
                }
            }
            steps {
                script {
                    // Ejecuta el análisis con las variables inyectadas desde Jenkins
                    withSonarQubeEnv('SonarQube') {
                        sh '''
                            echo "Ejecutando análisis con SonarQube..."
                            sonar-scanner \
                              -Dsonar.projectKey=dvwa-sast-sonarqube \
                              -Dsonar.sources=. \
                              -Dsonar.token=$SONAR_AUTH_TOKEN
                        '''
                    }
                }
            }
            post {
                always {
                    // Descarga el reporte JSON desde la API REST de SonarQube
                    script {
                        echo "Descargando reporte desde SonarQube API..."
                        sh '''
                            curl -s -u $SONAR_AUTH_TOKEN: \
                                "$SONAR_HOST_URL/api/issues/search?componentKeys=dvwa-sast-sonarqube" \
                                -o sonar-report.json || echo "{}" > sonar-report.json
                        '''
                    }
                    // Archiva el reporte como artefacto
                    archiveArtifacts artifacts: 'sonar-report.json', fingerprint: true
                    echo "Reporte de SonarQube archivado correctamente en Jenkins."
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    echo "Esperando resultado de la Quality Gate..."
                    timeout(time: 2, unit: 'MINUTES') {
                        waitForQualityGate abortPipeline: true
                    }
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
