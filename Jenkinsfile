pipeline {
    agent any
    tools {
        jdk 'jdk21'
        maven 'maven3'
    }
    parameters {
      string(name: 'sonar_IP', defaultValue: '100.54.191.181', description: 'IP of sonarqube')
      string(name: 'nexus_IP', defaultValue: '3.107.227.103', description: 'IP of nexus')
    }
    environment {
      SONARQUBE_URL="http://${params.sonar_IP}:9000"
      SONARQUBE_TOKEN=credentials('Sonar-token')
      NEXUS_URL="http://${params.nexus_IP}:8081"
      NEXUS_REPO='maven-releases'
      APP_EC2_IP='3.107.169.163'
      SSH_CRED_ID='ec2-ssh'
    }
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'master',
                url: 'https://github.com/ShilpaLB28/webapp.git'
            }
        }
        stage('Sonarqube Analysis') {
            steps {
                dir('webapp') {
                    sh """
                    mvn sonar:sonar \
                    -Dsonar.projectKey=MavenProject \
                    -Dsonar.host.url=$SONARQUBE_URL \
                    -Dsonar.login=$SONARQUBE_TOKEN
                    """
                }
            }
        }
        stage('Build') {
            steps {
                dir('webapp') {
                sh 'mvn clean package -DskipTests'
                }
            }
        }
        stage('Upload to Nexus') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'nexus-cred',
                    usernameVariable: 'NEXUS_USER',
                    passwordVariable: 'NEXUS_PASS'
                )]) {
                    dir('webapp') {
                        sh """
                        mvn deploy -DskipTests \
                        -DaltDeploymentRepository=nexus::default::$NEXUS_URL/repository/${NEXUS_REPO}/ \
                        -Dnexus.username=$NEXUS_USER -Dnexus.password=$NEXUS_PASS
                        """
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                sshagent(['ec2-key']) {
                    sh """
                    # Pull artifact from Nexus
                    ssh -o StrictHostKeyChecking=no ec2-user@${APP_EC2_IP} '
                      wget $NEXUS_URL/repository/${NEXUS_REPO}/com/example/maven-project/maven-project/1.0-SNAPSHOT/maven-project-1.0-SNAPSHOT.war
                      sudo mv /tmp/app.war /opt/tomcat/webapps/
                      sudo systemctl restart tomcat
                    '
                    """
                }
            }
        }
    }
}