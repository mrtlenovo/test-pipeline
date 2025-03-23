pipeline {
    agent any

    environment {
        NEXUS_CREDENTIALS = credentials('nexus-creds-id')
        SONARQUBE_SERVER = 'sonarqube-server'
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
                sh 'mvn clean package'
            }
        }

        stage('Publish to Nexus') {
            steps {
                sh '''
                curl -v -u $NEXUS_CREDENTIALS -X PUT --upload-file target/your-app.jar \
                http://nexus.cicd-prod.svc.cluster.local:8081/repository/maven-releases/your-app.jar
                '''
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
            junit '**/target/surefire-reports/*.xml'
            archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
        }
    }
}
