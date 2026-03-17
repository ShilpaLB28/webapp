pipeline {
    agent any
    tools {
        jdk 'jdk21'
        maven 'maven3'
    }
    parameters {
      string(name: 'sonar_IP', defaultValue: '100.54.191.181', description: 'IP of sonarqube')
      string(name: 'nexus_IP', defaultValue: '3.107.227.103', description: 'IP of nexus')
      string(name: 'deploy_IP', defaultValue:'3.107.169.163 ', description: 'IP of Deploy Server')  
    }
    environment {
      SONARQUBE_URL="http://${params.sonar_IP}:9000"
      SONARQUBE_TOKEN=credentials('Sonar-token')
      NEXUS_URL="http://${params.nexus_ip}:8081/repository/maven-releases/"
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
                    sh '''
                    mvn clean deploy -DskipTests \
                    -DaltDeploymentRepository=nexus-releases::default::${NEXUS_URL}
                    '''
                }
            }
        }
        stage('Deploy') {
            steps {
                sshagent(credentials:['ec2-key']) {
                    sh '''
                    scp -o StrictHostKeyChecking=no \
                        webapp/target/webapp.war \
                        ubuntu@${params.deploy_IP}:/tmp/webapp.war
                        
                    ssh -o StrictHostKeyChecking=no ubuntu@${params.deploy_IP} "
                        sudo cp /tmp/webapp.war /var/lib/tomcat9/webapps/webapp.war &&
                        sudo systemctl restart tomcat9
                    "
                    '''
                }
            }
        }
    }
}