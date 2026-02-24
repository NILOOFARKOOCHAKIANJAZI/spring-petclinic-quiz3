pipeline {
    agent any

    // Run every 5 minutes on Mondays (Jenkins: 1 = Monday)
    triggers {
        cron('H/5 * * * 1')
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
                // Make mvnw executable (helps on Linux agents)
                sh 'chmod +x mvnw || true'
                sh './mvnw -B clean test'
            }
            post {
                always {
                    // Publish JUnit test results in Jenkins (optional but nice)
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('JaCoCo Code Coverage') {
            steps {
                // Generates JaCoCo report
                // prepare-agent + test creates jacoco.exec, then report creates HTML in target/site/jacoco
                sh './mvnw -B jacoco:prepare-agent test jacoco:report'
            }
            post {
                always {
                    // Archive the HTML report so you can screenshot it
                    archiveArtifacts artifacts: 'target/site/jacoco/**', allowEmptyArchive: true
                }
            }
        }

        stage('Package Artifact (JAR)') {
            steps {
                // Package artifact (jar). Skip tests here because we already tested above.
                sh './mvnw -B package -DskipTests'
            }
            post {
                success {
                    // Archive jar artifact for proof
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
            }
        }
    }
}
