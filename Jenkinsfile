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
                    env.appVersion = packageJson.version
                    echo "Building version ${env.appVersion}"
                }
            }
        }

        stage('Install dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool name: 'sonar-8'

                    withSonarQubeEnv('sonar-server') {
                        sh "${scannerHome}/bin/sonar-scanner"
                    }
                }
            }
        }

        stage('Build Image') {
            steps {
                withAWS(credentials: 'aws-creds', region: "${region}") {
                    sh """
                        aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${ACC_ID}.dkr.ecr.${region}.amazonaws.com
                        docker build -t ${ACC_ID}.dkr.ecr.${region}.amazonaws.com/roboshop/catalogue:${env.appVersion} .
                        docker push ${ACC_ID}.dkr.ecr.${region}.amazonaws.com/roboshop/catalogue:${env.appVersion}

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