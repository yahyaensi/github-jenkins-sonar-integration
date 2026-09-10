pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()

        buildDiscarder(
            logRotator(
                numToKeepStr: '20',
                artifactNumToKeepStr: '10'
            )
        )
    }

    environment {
        // Nom de l'installation SonarQube configurée dans Jenkins
        SONARQUBE_ENV = 'SonarQube'

        // Nom de ton projet SonarQube
        SONAR_PROJECT_KEY = 'mon-projet'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Tests') {
            steps {
                sh '''
                    ./mvnw clean verify
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {

                    sh '''
                        ./mvnw sonar:sonar \
                          -Dsonar.projectKey=${SONAR_PROJECT_KEY}
                    '''
                }
            }
        }

        stage('SonarQube Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {

                    waitForQualityGate(
                        abortPipeline: true
                    )
                }
            }
        }
    }

    post {

        success {
            echo "✅ Build + tests + SonarQube : SUCCESS"
        }

        failure {
            echo "❌ Build ou Quality Gate : FAILURE"
        }

        unstable {
            echo "⚠️ Build : UNSTABLE"
        }

        always {
            echo "Build terminé : ${currentBuild.currentResult}"
            echo "Branche : ${env.BRANCH_NAME}"

            script {
                if (env.CHANGE_ID) {
                    echo "Pull Request : #${env.CHANGE_ID}"
                    echo "Target branch : ${env.CHANGE_TARGET}"
                }
            }
        }
    }
}