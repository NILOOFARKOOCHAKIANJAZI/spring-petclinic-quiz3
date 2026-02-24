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

        stage('Build + Tests + JaCoCo Report') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'chmod +x mvnw || true'
                        // Single run: tests + jacoco report
                        sh './mvnw -B clean jacoco:prepare-agent test jacoco:report'
                    } else {
                        bat 'mvnw.cmd -B clean jacoco:prepare-agent test jacoco:report'
                    }
                }
            }
            post {
                always {
                    // Test results in Jenkins
                    junit testResults: 'target/surefire-reports/*.xml', allowEmptyResults: true

                    // Publish JaCoCo coverage in Jenkins (requires "JaCoCo" plugin)
                    jacoco(
                        execPattern: 'target/jacoco.exec',
                        classPattern: 'target/classes',
                        sourcePattern: 'src/main/java',
                        exclusionPattern: '**/target/**'
                    )

                    // Keep HTML report as build artifact (so you can open index.html)
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
