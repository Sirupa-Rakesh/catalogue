pipeline {
    agent {
        node {
            label 'roboshop'
        }
    }

    environment {
        appVersion = ""
        ACC_ID = "489693027985"
        region = "us-east-1"
    }

    options {
        timeout(time: 5, unit: 'MINUTES')
    }

    stages {

        stage('Read version') {
            steps {
                script {
                    def packageJson = readJSON file: 'package.json'
                    appVersion = packageJson.version
                    echo "Building version ${appVersion}"
                }
            }
        }

        stage('Install dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build Image') {
            steps {
                withAWS(credentials: 'aws-creds', region: "${region}") {
                        // Commands here have AWS authentication
                        sh """
                            aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 
                            ${ACC_ID}.dkr.ecr.${region}.amazonaws.com
                            docker build -t ${ACC_ID}.dkr.${region}.amazonaws.com/roboshop/catalogue:latest:${appVersion} .
                            docker push${ACC_ID}.dkr.ecr.${region}.amazonaws.com/roboshop/catalogue:latest:${appVersion} 

                        """
            }
        }
    }

    post {
        always {
            echo 'I will always say Hello again!'
        }

        success {
            echo 'pipeline success'
        }

        failure {
            echo 'pipeline failure'
        }
    }
}