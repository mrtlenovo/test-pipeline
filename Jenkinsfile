pipeline {
    agent any

    tools {
        maven 'maven'    // Replace 'maven' with the exact Maven installation name in Global Tool Configuration
        jdk 'jdk11'      // Replace 'jdk11' with the correct JDK installation name in Jenkins
    }

    environment {
        NEXUS_CREDENTIALS = credentials('nexus-creds')  // ID from Jenkins credentials
        SONARQUBE_SERVER = 'SonarQube'                  // SonarQube server configuration in Jenkins
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/mrtlenovo/test-pipeline.git'
            }
        }

        stage('Code Quality Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_SERVER}") {  // SonarQube integration
                    sh 'mvn clean verify sonar:sonar'
                }
            }
        }

        stage('Build and Package') {
            steps {
                sh 'mvn clean package -DskipTests'  // Build application without running tests
            }
        }

        stage('Publish to Nexus') {
            steps {
                // Publish artifact to Nexus using credentials from Jenkins environment
                sh """
                mvn deploy -DskipTests \
                    -Dnexus.username=${NEXUS_CREDENTIALS_USR} \
                    -Dnexus.password=${NEXUS_CREDENTIALS_PSW}
                """
            }
        }

        stage('Deploy to OpenShift') {
            steps {
                // Switch to correct OpenShift project and trigger deployment
                sh 'oc project cicd-prod'
                sh 'oc start-build your-app --from-file=target/your-app.jar --wait'
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true  // Archive JAR file
            junit '**/target/surefire-reports/*.xml'                          // Publish test results
        }
    }
}
