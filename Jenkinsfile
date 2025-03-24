pipeline {
    agent any

    environment {
        NEXUS_CREDENTIALS = credentials('nexus-creds') // ID from Jenkins credentials
        SONARQUBE_SERVER = 'SonarQube'  // Name of your SonarQube server from Jenkins system config
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/mrtlenovo/test-pipeline.git'
            }
        }

        stage('Code Quality Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {  // Using SonarQube integration
                    sh 'mvn clean verify sonar:sonar'
                }
            }
        }

        stage('Build and Package') {
            steps {
                sh 'mvn clean package -DskipTests'  // Build your app (JAR or WAR)
            }
        }

        stage('Publish to Nexus') {
            steps {
                // Deploy the generated JAR/WAR file to Nexus
                sh """
                mvn deploy -DskipTests \
                    -Dnexus.username=${NEXUS_CREDENTIALS_USR} \
                    -Dnexus.password=${NEXUS_CREDENTIALS_PSW}
                """
            }
        }

        stage('Deploy to OpenShift') {
            steps {
                // Deploy the artifact to your OpenShift cluster
                sh 'oc project cicd-prod'
                sh 'oc start-build your-app --from-file=target/your-app.jar --wait'
            }
        }
    }

    post {
        always {
            // Archive the JAR file and publish test results
            archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
            junit '**/target/surefire-reports/*.xml'
        }
    }
}
