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
    }
}
