pipeline {
    agent {
        label 'roboshop-dev'
    }

    environment {
        appVersion = ''
        REGION  = "us-east-1"
        ACC_ID  = "108717859359"
        PROJECT =  "roboshop"
        COMPONENT = "catalogue"
    }

    options {
        timeout(time: 30, unit: 'SECONDS')
    }

    /* parameters {
        string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')
        text(name: 'BIOGRAPHY', defaultValue: '', description: 'Enter some information about the person')
        booleanParam(name: 'TOGGLE', defaultValue: true, description: 'Toggle this value')
        choice(name: 'CHOICE', choices: ['One', 'Two', 'Three'], description: 'Pick something')
        password(name: 'PASSWORD', defaultValue: 'SECRET', description: 'Enter a password')
    } */

    stages {
        
    stage('Read package.json') {
            steps {
                script {
                    def packageJson = readJSON file: 'package.json'
                    appVersion = packageJson.version
                    echo "Package version: ${appVersion}"
                }
            }
        }
        stage("Install Dependencies") {
            steps {
                script {
                    sh """
                        npm install
                        """
                }
            }
        }

       stage("Docker Build") {
            steps {
                script {
                    withAWS(credentials: 'jenkins-ecr-user', region: 'us-east-1') {
                        sh """
                        aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com
                        docker build -t  ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion} .
                        docker push ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion}
                        """
                      
                    }
                }

            }
       }

        stage('Parameters') {
            steps {
                script {
                    echo "Hello ${params.PERSON}"
                    echo "Biography: ${params.BIOGRAPHY}"
                    echo "Toggle: ${params.TOGGLE}"
                    echo "Choice: ${params.CHOICE}"
                    echo "Password: ${params.PASSWORD}"
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    echo 'Deploying.....'
                }
            }
        }
    }

    post {
        always {
            echo "Say hello"
            deleteDir()
        }
        success {
            echo "pipeline is success"
        }
        failure {
            echo "pipeline is failure"
            }
        }
    }