pipeline {
    agent none

    tools {
        maven 'apache-maven-3.9.10'
    }

    options {
        skipDefaultCheckout(true) // Prevent auto-checkout on all nodes
    }

    environment {
        TOMCAT_HOME = '/mnt/server/apache-tomcat-10.1.42'
        DB_URL = 'database-1.cbqy4wmkgmg5.ap-south-1.rds.amazonaws.com'
        DB_USER = 'admin'
        DB_PASS = 'admin123'
        DB_NAME = 'test'
    }

    stages {
        stage('Configure DB Connection') {
            agent { label 'built-in' }
            steps {
                dir("${env.WORKSPACE}/src/main/webapp") {
                    sh """
                        echo "Updating JDBC URL and credentials in userRegistration.jsp"
                        sed -i 's|jdbc:mysql://localhost:3306/test|jdbc:mysql://${DB_URL}:3306/${DB_NAME}|' userRegistration.jsp
                        sed -i 's|"root", "root"|"${DB_USER}", "${DB_PASS}"|' userRegistration.jsp
                        echo "Resulting DB line:"
                        grep DriverManager userRegistration.jsp
                    """
                }
            }
        }

        stage('Build WAR') {
            agent { label 'built-in' }
            steps {
                dir("${env.WORKSPACE}") {
                    sh 'mvn clean package'
                    stash name: 'warfile', includes: 'target/*.war'
                }
            }
        }

        stage('Deploy to Tomcat on Slave') {
            agent { label 'dev' }
            steps {
                unstash 'warfile'
                sh '''
                    echo "Deploying WAR to Tomcat on slave"
                    sudo cp target/LoginWebApp.war ${TOMCAT_HOME}/webapps/
                    sudo chmod -R 755 ${TOMCAT_HOME}/webapps

                    echo "Stopping Tomcat if running"
                    sudo ${TOMCAT_HOME}/bin/shutdown.sh || true
                    sleep 3

                    echo "Starting Tomcat"
                    sudo nohup ${TOMCAT_HOME}/bin/startup.sh > /dev/null 2>&1 &
                    sleep 5

                    echo "Verifying if Tomcat is listening on port 8080"
                    sudo netstat -tulnp | grep 8080 || echo "Tomcat failed to start"
                '''
            }
        }
    }
}
