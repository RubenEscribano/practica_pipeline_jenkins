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
                // sh 'make test-e2e'
                sh '''
                    set -e
                    mkdir -p results
                    docker network create calc-test-e2e || true
                    docker stop apiserver || true
                    docker rm --force apiserver || true
                    docker stop calc-web || true
                    docker rm --force calc-web || true

                    docker run -d --rm --volume $(pwd):/opt/calc --network calc-test-e2e \
                        --env PYTHONPATH=/opt/calc --name apiserver --env FLASK_APP=app/api.py \
                        -p 5001:5000 -w /opt/calc calculator-app:latest flask run --host=0.0.0.0

                    echo "Esperando a que el API responda..."
                    HEALTHY=0
                    for i in {1..20}; do
                        if docker run --rm --network calc-test-e2e curlimages/curl:8.5.0 -sSf http://apiserver:5000/ > /dev/null; then
                            HEALTHY=1
                            break
                        fi
                        sleep 2
                    done

                    if [ "$HEALTHY" -ne 1 ]; then
                        echo "El API no respondió al healthcheck."
                        docker logs apiserver || true
                        exit 1
                    fi

                    docker run -d --rm --volume $(pwd)/web:/usr/share/nginx/html \
                        --volume $(pwd)/web/constants.test.js:/usr/share/nginx/html/constants.js \
                        --volume $(pwd)/web/nginx.conf:/etc/nginx/conf.d/default.conf \
                        --network calc-test-e2e --name calc-web -p 80:80 nginx

                    docker run --rm --volume $(pwd)/test/e2e/cypress.json:/cypress.json \
                        --volume $(pwd)/test/e2e/cypress:/cypress \
                        --volume $(pwd)/results:/results --network calc-test-e2e \
                        cypress/included:4.9.0 --browser chrome \
                        --reporter junit --reporter-options "mochaFile=/results/e2e_result.xml,toConsole=false" || true
                '''
            }
            post {
                always {
                    sh '''
                        docker rm --force apiserver || true
                        docker rm --force calc-web || true
                        docker network rm calc-test-e2e || true
                        docker run --rm --volume $(pwd):/opt/calc --env PYTHONPATH=/opt/calc \
                            -w /opt/calc calculator-app:latest junit2html \
                            results/e2e_result.xml results/e2e_result.html || true
                    '''
                    junit allowEmptyResults: true, testResults: 'results/*e2e*.xml'
                    archiveArtifacts artifacts: 'results/*e2e*.xml', allowEmptyArchive: true
                    archiveArtifacts artifacts: 'test/e2e/cypress/screenshots/**/*', allowEmptyArchive: true
                    archiveArtifacts artifacts: 'test/e2e/cypress/videos/**/*', allowEmptyArchive: true
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
