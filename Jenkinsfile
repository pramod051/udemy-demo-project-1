pipeline {
    agent any

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        SONAR_TOKEN = credentials('sonar-token')
        SONAR_ORG = "pramod051"
        SONAR_PROJECT_KEY = "pramod051_project1"

        // Database ENV VARS from Jenkins Credentials
        DB_HOST = credentials('DB_HOST')
        DB_USER = credentials('DB_USER')
        DB_PASSWORD = credentials('DB_PASSWORD')
        DB_NAME = credentials('DB_NAME')
    }

    stages {

        stage('Code Analysis') {
            steps {
                withSonarQubeEnv('SonarCloud') {
                    sh """
                        ${SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.organization=${SONAR_ORG} \
                        -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=https://sonarcloud.io \
                        -Dsonar.login=${SONAR_TOKEN}
                    """
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    docker.withRegistry('', 'docker-cred') {
                        def image = docker.build("pramod051/udemy-demo-app1:latest")
                        image.push()
                    }
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                script {
                    sh '''
                    docker rm -f $(docker ps -q) || true

                    docker run -d -p 3000:3000 \
                      -e DB_HOST=${DB_HOST} \
                      -e DB_USER=${DB_USER} \
                      -e DB_PASSWORD=${DB_PASSWORD} \
                      -e DB_NAME=${DB_NAME} \
                      pramod051/udemy-demo-app1:latest
                    '''
                }
            }
        }
    }
}
