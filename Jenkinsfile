pipeline {
    agent none

    tools {
        maven 'apache-maven-3.9.10'
    }

    environment {
        DB_URL = 'database-1.cbqy4wmkgmg5.ap-south-1.rds.amazonaws.com'
        DB_USER = 'admin'
        DB_PASS = 'admin123'
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

        stage('Configure Database Connection') {
            agent { label 'built-in' }
            steps {
                dir('/mnt/project') {
                    sh '''
                    sed -i 's|jdbc:mysql://localhost:3306/test", "root", "root"|jdbc:mysql://database-1.cbqy4wmkgmg5.ap-south-1.rds.amazonaws.com:3306/loginwebapp", "admin", "admin123"|g' userRegistration.jsp
                    '''
                }
            }
        }

        stage('Deploy WAR on Tomcat Slave') {
            agent { label 'slave-1' }
            steps {
                dir('/mnt/project/target') {
                    unstash 'warfile'
                    sh '''
                    cp *.war ${TOMCAT_HOME}/webapps/
                    chmod -R 777 ${TOMCAT_HOME}
                    ${TOMCAT_HOME}/bin/startup.sh
                    '''
                }
            }
        }
    }
}
