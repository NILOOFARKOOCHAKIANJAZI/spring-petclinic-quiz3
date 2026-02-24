pipeline {
    agent any

    triggers {
        cron('H/5 * * * 1')   // every 5 minutes on Mondays
    }

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build + Unit Tests') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'chmod +x mvnw || true'
                        sh './mvnw -B clean test'
                    } else {
                        bat 'mvnw.cmd -B clean test'
                    }
                }
            }
            post {
                always {
                    // Don’t fail the whole build if reports are missing
                    junit testResults: 'target/surefire-reports/*.xml', allowEmptyResults: true
                }
            }
        }

        stage('JaCoCo Code Coverage') {
            steps {
                script {
                    if (isUnix()) {
                        sh './mvnw -B jacoco:prepare-agent test jacoco:report'
                    } else {
                        bat 'mvnw.cmd -B jacoco:prepare-agent test jacoco:report'
                    }
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'target/site/jacoco/**', allowEmptyArchive: true
                }
            }
        }

        stage('Package Artifact (JAR)') {
            steps {
                script {
                    if (isUnix()) {
                        sh './mvnw -B package -DskipTests'
                    } else {
                        bat 'mvnw.cmd -B package -DskipTests'
                    }
                }
            }
            post {
                success {
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
            }
        }
    }
}
