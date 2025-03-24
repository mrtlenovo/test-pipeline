pipeline {
    agent any

    tools {
        maven 'maven3'  // Update to the exact Maven name you configured in Global Tool Configuration
    }

    environment {
        NEXUS_CREDENTIALS = credentials('nexus-creds') 
        SONARQUBE_SERVER = 'SonarQube'  
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/mrtlenovo/test-pipeline.git'
            }
        }

        stage('Code Quality Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn clean verify sonar:sonar'
                }
            }
        }

        stage('Build and Package') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Publish to Nexus') {
            steps {
                sh """
                mvn deploy -DskipTests \
                    -Dnexus.username=${NEXUS_CREDENTIALS_USR} \
                    -Dnexus.password=${NEXUS_CREDENTIALS_PSW}
                """
            }
        }

        stage('Deploy to OpenShift') {
            steps {
                sh 'oc project cicd-prod'
                sh 'oc start-build your-app --from-file=target/your-app.jar --wait'
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
            junit '**/target/surefire-reports/*.xml'
        }
    }
}
