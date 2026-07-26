pipeline {
    agent {
        node {
            label 'roboshop'
        }
    }

    environment {
        appVersion = ""
        ACC_ID = "489693027985"    // Example: 123456789012
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
        stage('sonarqube analysis'){
            tools {
                sonar 'sonar-8'
            }
            steps {
                script {
                    sh "sonar-scanner"
                }
            }
        }

        stage('Build Image') {
            steps {
                withAWS(credentials: 'aws-creds', region: "${region}") {
                    sh """
                        aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${ACC_ID}.dkr.ecr.${region}.amazonaws.com
                        docker build -t ${ACC_ID}.dkr.ecr.${region}.amazonaws.com/roboshop/catalogue:${appVersion} .
                        docker push ${ACC_ID}.dkr.ecr.${region}.amazonaws.com/roboshop/catalogue:${appVersion}
                    """
                }
            }
        }

    }

    post {
        always {
            echo 'I will always say Hello again!'
        }

        success {
            echo 'Pipeline Success'
        }

        failure {
            echo 'Pipeline Failure'
        }
    }
}