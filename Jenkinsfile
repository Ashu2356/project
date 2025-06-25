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
        JAVA_HOME = '/usr/lib/jvm/java-1.8.0'
        PATH = "${JAVA_HOME}/bin:${env.PATH}"
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

        stage('Deploy WAR on Tomcat Slave') {
            agent { label 'slave-1' }
            steps {
                unstash name: 'warfile'
                dir('project/target') {
                    sh """
                        cp ${WAR_NAME} ${TOMCAT_HOME}/webapps/
                        chmod +x ${TOMCAT_HOME}/bin/*.sh
                        ${TOMCAT_HOME}/bin/shutdown.sh || true
                        ${TOMCAT_HOME}/bin/startup.sh
                    """
                }
            }
        }
    }
}
