pipeline {
    agent { label 'slave-1' }

    environment {
        NETWORK_NAME = 'my-network'
        MYSQL_CONTAINER = 'mysql-db'
        TOMCAT_CONTAINER = 'tomcat-app'
        DB_NAME = 'user'
        DB_USER = 'admin'
        DB_PASS = 'admin123'
        WAR_NAME = 'LoginWebApp.war'
    }

    tools {
        maven 'apache-maven-3.9.10'
    }

    stages {

        stage('Build WAR') {
            steps {
                sh 'mvn clean package'
                stash name: 'warfile', includes: "target/${env.WAR_NAME}"
            }
        }

        stage('Start Docker Network & Containers') {
            steps {
                sh '''
                    echo "Creating custom Docker bridge network if not exists"
                    docker network inspect ${NETWORK_NAME} >/dev/null 2>&1 || docker network create ${NETWORK_NAME}

                    echo "Starting MySQL container"
                    docker run -d --name ${MYSQL_CONTAINER} --network ${NETWORK_NAME} \
                      -e MYSQL_ROOT_PASSWORD=${DB_PASS} \
                      -e MYSQL_DATABASE=${DB_NAME} \
                      -e MYSQL_USER=${DB_USER} \
                      -e MYSQL_PASSWORD=${DB_PASS} \
                      mysql:8

                    echo "Waiting for MySQL to initialize..."
                    sleep 20

                    echo "Starting Tomcat container"
                    docker run -d --name ${TOMCAT_CONTAINER} --network ${NETWORK_NAME} \
                      -v $(pwd)/target/${WAR_NAME}:/usr/local/tomcat/webapps/${WAR_NAME} \
                      tomcat:9.0
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    echo "Checking if WAR is deployed in Tomcat"
                    sleep 10
                    curl -s http://localhost:8080/${WAR_NAME.replace('.war','')}/ || echo "App not responding"
                '''
            }
        }
    }

    post {
        always {
            echo 'Cleaning up containers (optional in real builds)'
            sh '''
                docker rm -f ${MYSQL_CONTAINER} ${TOMCAT_CONTAINER} || true
            '''
        }
    }
}
