pipeline {
    agent {
        label 'docker'
    }
    stages {
        stage('Source') {
            steps {
                git branch: 'main', url: 'https://github.com/RubenEscribano/actividad2-eiec.git'
            }
        }
        stage('Build') {
            steps {
                echo 'Building stage!'
                sh 'make build'
            }
        }
        stage('Unit tests') {
            steps {
                sh 'make test-unit'
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: 'results/*unit*.xml'
                    archiveArtifacts artifacts: 'results/*unit*.xml', allowEmptyArchive: true
                }
            }
        }
        stage('API tests') {
            steps {
                sh 'make test-api'
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: 'results/*api*.xml'
                    archiveArtifacts artifacts: 'results/*api*.xml', allowEmptyArchive: true
                }
            }
        }
        stage('E2E tests') {
            steps {
                sh 'make test-e2e'
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: 'results/*e2e*.xml'
                    archiveArtifacts artifacts: 'results/*e2e*.xml', allowEmptyArchive: true
                }
            }
        }
    }
    post {
        failure {
            mail to: 'rubenescribanomendiola@gmail.com',
                 subject: "Fallo en ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: """El pipeline ${env.JOB_NAME} (ejecución #${env.BUILD_NUMBER}) ha finalizado con errores.
Por favor, revise los informes de pruebas para más detalles."""
        }
        always {
            cleanWs()
        }
    }
}
