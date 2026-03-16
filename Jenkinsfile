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
      Nexus_URL="http://${params.nexus_IP}:8081"
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
                dir('webapp') {
                    sh 'mvn clean deploy -DskipTests'
                }
            }
        }
        stage('Deplo') {
            steps {
                sshagent(['ec2-key']) {
                    sh '''
                    ssh ubuntu@3.107.169.163 "
                    wget $NEXUS_URL/repository/maven-release/com/example/maven-project/webapp/1.0-SNAPSHOT/webapp.war
                    sudo cp webapp.war ~/tomcat/webapps/
                    ~/tomcat/bin/shutdown.sh
                    ~/tomcat/bin/startup.sh
                    "
                    '''
                }
            }
        }
    }
}