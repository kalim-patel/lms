pipeline {
    agent any
    environment {
        // More detail: 
        // https://jenkins.io/doc/book/pipeline/jenkinsfile/#usernames-and-passwords
        NEXUS_CRED = credentials('nexus')
   }

    stages {
        stage('Build') {
            steps {
                echo 'Building..'
                sh 'cd webapp && npm install && npm run build'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing..'
                sh 'cd webapp && sudo docker container run --rm -e SONAR_HOST_URL="http://13.50.224.173:9000" -e SONAR_TOKEN="sqp_73ba242fb9199cd2c07e6dec3a8120acd4f471ea" -v ".:/usr/src" sonarsource/sonar-scanner-cli -Dsonar.projectKey=lms'
            }
        }
        stage('Release') {
            steps {
                echo 'Release Nexus'
                sh 'rm -rf *.zip'
                sh 'cd webapp && zip dist-${BUILD_NUMBER}.zip -r dist'
                sh 'cd webapp && curl -v -u $NEXUS_CRED_USR:$NEXUS_CRED_PSW --upload-file dist-${BUILD_NUMBER}.zip http://13.50.224.173:8081/repository/lms/'
            }
        }
        stage('Deploy LMS') {
            steps {
                echo 'Deploying LMS'

                sh 'curl -v -u $NEXUS_CRED_USR:$NEXUS_CRED_PSW -o lms.zip http://13.50.224.173:8081/repository/lms/dist-${BUILD_NUMBER}.zip'

                sh 'sudo rm -rf /var/www/html/*'

                sh 'sudo unzip -o lms.zip -d /tmp/lms'

                sh 'sudo cp -r /tmp/lms/webapp/dist/* /var/www/html/'
            }
        }
        stage('Clean Up Workspace') {
            steps {
                echo 'Cleaning Work Space'
                cleanWs()
            }
        }
    }
}