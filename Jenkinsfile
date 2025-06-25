pipeline {
  agent none

  tools {
    maven 'apache-maven-3.9.10'
  }

  environment {
    WAR_NAME = 'LoginWebApp.war'
    TOMCAT_HOME = '/mnt/apache-tomcat-10.1.42'
    DB_URL = 'database-1.cbqy4wmkgmg5.ap-south-1.rds.amazonaws.com'
    DB_USER = 'admin'
    DB_PASS = 'admin123'
    DB_NAME = 'test'
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

    stage('Configure DB Connection') {
      agent { label 'built-in' }
      steps {
        dir('/mnt/project/src/main/webapp') {
          sh '''
            sed -i 's|jdbc:mysql://.*:3306/.*", ".*", ".*"|jdbc:mysql://${DB_URL}:3306/${DB_NAME}", "${DB_USER}", "${DB_PASS}"|g' userRegistration.jsp
          '''
        }
      }
    }

    stage('Build WAR with Maven') {
      agent { label 'built-in' }
      steps {
        dir('/mnt/project') {
          sh 'mvn clean package'
          stash name: 'warfile', includes: 'target/*.war'
        }
      }
    }

    stage('Deploy WAR to Tomcat (Slave)') {
      agent { label 'slave-1' }
      steps {
        dir("${TOMCAT_HOME}/webapps") {
          unstash 'warfile'
          sh '''
            echo "Stopping Tomcat..."
            ${TOMCAT_HOME}/bin/shutdown.sh || echo "Tomcat not running"
            sleep 3
            echo "Deploying WAR..."
            cp /mnt/slave/workspace/my-web-app/target/${WAR_NAME} .
            echo "Starting Tomcat..."
            ${TOMCAT_HOME}/bin/startup.sh
          '''
        }
      }
    }
  }
}
