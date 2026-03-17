pipeline {
    agent any
    tools {
        jdk 'jdk21'
        maven 'maven3'
    }
    parameters {
      string(name: 'sonar_IP', defaultValue: '52.62.89.26', description: 'IP of sonarqube')
      string(name: 'nexus_IP', defaultValue: '3.107.227.103', description: 'IP of nexus')
      string(name: 'deploy_IP', defaultValue:'3.107.169.163', description: 'IP of Deploy Server')  
    }
    environment {
      SONARQUBE_URL="http://${params.sonar_IP}:9000"
      SONARQUBE_TOKEN=credentials('Sonar-token')
      NEXUS_URL="http://${params.nexus_IP}:8081"
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
                    echo "NEXUS_URL=$NEXUS_URL"

                    mvn clean deploy -DskipTests \
                    -DaltDeploymentRepository=nexus-snapshots::default::http://3.107.227.103:8081/nexus/content/repositories/maven-snapshots/
                    '''
                }
            }
        }
        stage('Deploy') {
            steps {
                sshagent(credentials:['ec2-ssh']) {
                    sh '''
                    scp -o StrictHostKeyChecking=no \
                        webapp/target/webapp.war \
                        ubuntu@${deploy_IP}:/tmp/webapp.war

                    ssh -o StrictHostKeyChecking=no ubuntu@${deploy_IP} "
                        sudo cp /tmp/webapp.war /home/ubuntu/tomcat/webapps/webapp.war &&
                        /home/ubuntu/tomcat/bin/shutdown.sh
                        /home/ubuntu/tomcat/bin/startup.sh
                    "
                    '''
                }
            }
        }
    }
}