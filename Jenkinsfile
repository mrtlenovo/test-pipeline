pipeline {
    agent any

    tools {
        maven 'maven3'  // Ensure this matches the exact Maven name you configured in Jenkins Global Tool Configuration
    }

    environment {
        NEXUS_CREDENTIALS = credentials('nexus-creds')  // Credentials ID for Nexus
        SONARQUBE_SERVER = 'SonarQube'  // SonarQube server name in Jenkins
        JAVA_HOME = "/etc/pki/ca-trust/extracted/java"  // Java truststore path
        CERT_PATH = "/var/lib/jenkins/sonarqube_cert.pem"  // Path to the SonarQube certificate
    }

    stages {
        stage('Import SonarQube Certificate') {
            steps {
                script {
                    echo "Importing SonarQube certificate into Java truststore..."
                    sh '''
                    keytool -import -trustcacerts -keystore /var/lib/jenkins/jenkins-truststore.jks \
                            -storepass changeit -noprompt -alias sonarqube-cert \
                            -file $CERT_PATH
                    '''
                    echo "Certificate import completed successfully."
                }
            }
        }

        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/mrtlenovo/test-pipeline.git'
            }
        }

        stage('Code Quality Analysis') {
            steps {
                withEnv(["JAVA_OPTS=-Djavax.net.ssl.trustStore=/var/lib/jenkins/jenkins-truststore.jks -Djavax.net.ssl.trustStorePassword=changeit"]) {
                    withSonarQubeEnv('SonarQube') {  // SonarQube environment binding
                        sh 'mvn clean verify sonar:sonar'
                    }
                }
            }
        }

        stage('Build and Package') {
            steps {
                sh 'mvn clean package -DskipTests'  // Build the project without running tests
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
                sh 'oc project cicd-prod'  // Switch to the OpenShift project namespace
                sh 'oc start-build your-app --from-file=target/your-app.jar --wait'  // Start the OpenShift build with the packaged JAR
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true  // Archive the built JAR file
            junit '**/target/surefire-reports/*.xml'  // Publish JUnit test results
        }
    }
}
