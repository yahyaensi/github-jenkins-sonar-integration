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
        SONAR_PROJECT_KEY = 'github-jenkins-sonar-integration'
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
                // Le bloc script permet d'utiliser du code Groovy (comme def)
                script {
                    def scannerHome = tool 'SonarScanner'
                    
                    withSonarQubeEnv("${SONARQUBE_ENV}") {
                        sh "${scannerHome}/bin/sonar-scanner \
                          -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                          -Dsonar.sources=src/main/java \
                          -Dsonar.java.binaries=target/classes"
                    }
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