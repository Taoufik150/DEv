pipeline {

    agent any

    tools {
        nodejs 'NodeJS-22'
    }

    options {
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 30, unit: 'MINUTES')
        timestamps()
    }

    environment {
        COMPOSE_PROJECT = 'tp-app'
        DB_USER         = 'tpuser'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Check Environment') {
            steps {
                sh '''
                    java -version
                    mvn -version
                    node -v
                    npm -v
                    docker --version
                    docker compose version
                '''
            }
        }

        stage('Backend - Build & Test') {
            steps {
                dir('backend') {
                    sh 'mvn -B clean verify'
                }
            }
            post {
                always {
                    junit allowEmptyResults: true,
                          testResults: 'backend/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Frontend - Install Dependencies') {
            steps {
                dir('frontend') {
                    sh 'npm ci'
                }
            }
        }

        stage('Frontend - Build') {
            steps {
                dir('frontend') {
                    sh 'npm run build'
                }
            }
        }

        stage('Archive') {
            steps {
                archiveArtifacts artifacts: 'backend/target/*.jar', fingerprint: true
            }
        }

        stage('Deploy (Docker Compose)') {
            when {
                branch 'main'
            }
            steps {
                withCredentials([string(credentialsId: 'db-password', variable: 'DB_PASSWORD')]) {
                    sh 'docker compose -p $COMPOSE_PROJECT up -d --build'
                }
            }
        }
    }

    post {
        success {
            echo "CI SUCCESS sur ${env.BRANCH_NAME} : Backend et Frontend compiles !"
        }

        failure {
            echo "CI FAILURE sur ${env.BRANCH_NAME} : Une erreur est survenue !"
        }

        cleanup {
            cleanWs()
        }
    }
}
