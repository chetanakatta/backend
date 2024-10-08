pipeline {

    agent{
        label 'AGENT-1' // our agent name
    }

    options{

        //after particular time job will be failed (timeout counter starts before agent is allocated)
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds() //disables multiple executions
        ansiColor('xterm')

    }

    parameters{

        booleanParam(name: 'deploy', defaultValue: false, description: 'Toggle this value')
    }

    environment{

        def appVersion = ''  //variable declaration
        nexusUrl = 'nexus.expense.fun:8081'

    }

    stages{

        stage ('read the version'){ 
            steps {
                script {
                    def packageJson = readJSON file: 'package.json'
                    appVersion = packageJson.version
                    echo "application version: $appVersion"
                }
            }
        }

        stage ('Install Dependencies'){
            steps {    
                sh """
                npm install
                ls -ltr
                echo "application version: $appVersion"
                """
            }    
        }

        stage ('Build'){
            steps {
                sh """
                zip -q -r backend-${appVersion}.zip * -x Jenkinsfile -x backend-${appVersion}.zip
                ls -ltr
                """
            }
        }

        stage ('Sonar Scan'){
            environment {
                scannerHome = tool 'sonar'  //referring scanner CLI
            }
            steps {
                script {
                    withSonarQubeEnv('sonar') { //reffering sonar server
                    sh "${scannerHome}/bin/sonar-scanner"
                    }
                }
            }
        }

        stage ('Quality Gate'){
            steps{
                timeout(time: 30, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }        
            }
        }

        stage ('Nexus Artifact Upload'){
            steps {
                script {
                    nexusArtifactUploader(
                       nexusVersion: 'nexus3',
                       protocol: 'http',
                       nexusUrl: "${nexusUrl}",
                       groupId: 'com.expense',
                       version: "${appVersion}",
                       repository: "backend",
                       credentialsId: 'nexus-auth',
                       artifacts: [
                        [artifactId: "backend",
                        classifier: '',
                        file: "backend-" + "${appVersion}" + '.zip',
                        type: 'zip']
                       ]

                    )

                }
            }
        }

        stage ('Deploy'){
            when{
                expression{
                    params.deploy
                }
            }
            steps{
                script{
                    def params = [
                        string(name: 'appVersion', value: "${appVersion}")
                    ]
                    build job: 'backend-deploy', parameters: params, wait: false
                }

            }
        }

    }    
    post{ //useful as alert for success or failure

        always{
            echo 'I will always say hello'
            deleteDir()    //to delete workspace after build
        }

        success{
            echo 'I will run when pipeline is success'
        }

        failure{
            echo 'I will run when pipeline is failure'
            // we configure with slack, when failed we get messege
        }
    }
}