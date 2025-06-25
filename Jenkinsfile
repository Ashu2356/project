pipeline {
  agent none

  tools {
    maven 'apache-maven-3.9.10'
  }

  environment {
    WAR_NAME = 'LoginWebApp.war'
    TOMCAT_HOME = '/mnt/apache-tomcat-10.1.42'
  }

  stages {

    stage('Clone Git Repo') {
      agent { label 'built-in' }
      steps {
        dir('/mnt/project') {
          checkout scm
        }
      }
    }

    stage('Build WAR with Maven') {
      agent { label 'built-in' }
      steps {
        dir('/mnt/project') {
          sh 'rm -rf ~/.m2/repository'
          sh 'mvn clean package'
          stash name: 'warfile', includes: 'target/*.war'
        }
      }
    }

    stage('Deploy WAR to Tomcat (Slave)') {
      agent { label 'slave-1' }
      steps {
        withEnv(["WAR_NAME=LoginWebApp.war", "TOMCAT_HOME=/mnt/apache-tomcat-10.1.42"]) {
          dir("${TOMCAT_HOME}/webapps") {
            unstash 'warfile'
            sh '''
              echo "Stopping Tomcat..."
              $TOMCAT_HOME/bin/shutdown.sh || echo "Tomcat not running"
              sleep 3
              echo "Copying WAR..."
              cp /mnt/slave/workspace/my-web-app/target/$WAR_NAME .
              echo "Starting Tomcat..."
              $TOMCAT_HOME/bin/startup.sh
            '''
          }
        }
      }
    }
  }
}
