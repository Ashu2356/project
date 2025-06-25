pipeline {
  agent none

  tools {
    maven 'apache-maven-3.9.10'
  }

  environment {
    DB_URL = 'database-1.cti0iqs4ugbm.ap-south-1.rds.amazonaws.com'
    DB_USER = 'admin'
    DB_PASS = '123456'
    WAR_NAME = 'LoginWebApp.war'
    TOMCAT_PATH = '/mnt/apache-tomcat-10.1.42'
  }

  stages {

    stage('Clone Repo') {
      agent { label 'built-in' }
      steps {
        dir('/mnt/project') {
          checkout scm
        }
      }
    }

    stage('Build WAR') {
      agent { label 'built-in' }
      steps {
        dir('/mnt/project') {
          sh 'mvn clean package'
          stash name: 'warfile', includes: 'target/*.war'
        }
      }
    }

    stage('Update DB Config') {
      agent { label 'built-in' }
      steps {
        dir('/mnt/project/src/main/webapp') {
          sh """
          sed -i 's|jdbc:mysql://localhost:3306/test", "root", "root"|jdbc:mysql://${DB_URL}:3306/loginwebapp", "${DB_USER}", "${DB_PASS}"|g' userRegistration.jsp
          """
        }
        dir('/mnt/project') {
          sh 'mvn clean package'
          stash name: 'warfile', includes: 'target/*.war'
        }
      }
    }

    stage('Deploy to Tomcat') {
      agent { label 'slave-1' }
      steps {
        unstash 'warfile'
        sh """
          cp target/${WAR_NAME} ${TOMCAT_PATH}/webapps/
          ${TOMCAT_PATH}/bin/shutdown.sh || true
          ${TOMCAT_PATH}/bin/startup.sh
        """
      }
    }
  }
}
