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
        timeout(time: 15, unit: 'MINUTES')
    }
 
     parameters {
        booleanParam(name: 'deploy', defaultValue: false, description: 'Toggle this value')
        
    } 

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
        stage ("Unit Testing") {
            steps {
                script {
                    sh """
                        echo "unit test"
                        """
                }
            }
        }
        stage ("Sonar Scanner") {
             environment {
                scannerHome = tool 'sonar-7.2'
            }
            steps {
                script {
                    withSonarQubeEnv(installationName: 'sonar-7.2') {
                        sh "${scannerHome}/bin/sonar-scanner"
                    }
                }
            }
        }
        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: false }
            }
        }
        stage('Dependabot Security Check') {
      steps {
        script {
          def response = sh(
            script: """curl -s -L \
              -H "Accept: application/vnd.github+json" \
              -H "Authorization: token ${GITHUB_TOKEN}" \
              -H "X-GitHub-Api-Version: 2022-11-28" \
              https://api.github.com/repos/KodukG01/catalogue/dependabot/alerts""",
            returnStdout: true
          ).trim()

          // Parse JSON
          def alerts = readJSON text: response

          // Filter for High and Critical
          def highCritical = alerts.findAll { alert ->
            def sev = alert.security_vulnerability.severity.toLowerCase()
            return sev == "high" || sev == "critical"
          }

          if (highCritical.size() > 0) {
            echo "❌ Found ${highCritical.size()} High/Critical security alerts!"
            highCritical.each { alert ->
              echo "  - ${alert.dependency.package.name} (${alert.security_vulnerability.severity}) : ${alert.security_advisory.summary}"
              echo "    Fix Version: ${alert.security_vulnerability.first_patched_version.identifier}"
              echo "    More Info: ${alert.html_url}"
            }
            error("Build failed due to open High/Critical security alerts.")
          } else {
            echo "✅ No High/Critical alerts found."
          }
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
       stage ("Trigger Deploy") {
        when {
            expression { params.deploy }
        }
        steps {
            script {
                build job: 'catalogue-cd',
                parameters: [
                              string(name: 'appVersion', value: "${appVersion}"),
                              string(name: 'deploy_to', value: 'dev')
                            ],
                propagata: false
                wait: false

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