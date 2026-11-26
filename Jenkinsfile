pipeline {
    agent any

    options {
        skipDefaultCheckout()
    }

    triggers {
        pollSCM('H/5 * * * *')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Backend - tests et JAR') {
            steps {
                dir('backend') {
                    sh 'mvn -B -DfailIfNoTests=true clean verify'
                }
            }
            post {
                always {
                    junit testResults: 'backend/target/surefire-reports/TEST-*.xml',
                          allowEmptyResults: true
                }
            }
        }

        stage('Frontend - build Angular') {
            steps {
                dir('frontend') {
                    sh 'npm ci'
                    sh 'npm run build'
                }
            }
        }

        stage('Archivage') {
            steps {
                archiveArtifacts artifacts: 'backend/target/*.jar', fingerprint: true
                archiveArtifacts artifacts: 'frontend/dist/frontend/**/*', fingerprint: true
            }
        }

        stage('Docker - construire les images') {
            steps {
                sh 'docker build -t "atelier4-backend:jenkins-$BUILD_NUMBER" ./backend'
                sh 'docker build -t "atelier4-frontend:jenkins-$BUILD_NUMBER" ./frontend'
            }
        }

        stage('Docker - publier les images') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-atelier4',
                    usernameVariable: 'DOCKERHUB_USER',
                    passwordVariable: 'DOCKERHUB_TOKEN'
                )]) {
                    sh '''
                        set +x
                        DOCKER_CONFIG="$(mktemp -d)"
                        export DOCKER_CONFIG
                        trap 'rm -f "$DOCKER_CONFIG/config.json"; rmdir "$DOCKER_CONFIG" || true' EXIT

                        printf '%s' "$DOCKERHUB_TOKEN" |
                            docker login --username "$DOCKERHUB_USER" --password-stdin

                        repo="islemsekrani/atelier4-app"
                        backend_image="$repo:backend-jenkins-$BUILD_NUMBER"
                        frontend_image="$repo:frontend-jenkins-$BUILD_NUMBER"

                        docker tag "atelier4-backend:jenkins-$BUILD_NUMBER" "$backend_image"
                        docker tag "atelier4-frontend:jenkins-$BUILD_NUMBER" "$frontend_image"

                        docker push "$backend_image"
                        docker push "$frontend_image"
                    '''
                }
            }
        }
    }
}
