pipeline {

    agent {
        label 'AGENT-1' // our agent name
    }

    options {
        //after particular time job will be failed (timeout counter starts before agent is allocated)
        timeout (time: 30, unit: 'MINUTES')
        disableConcurrentBuilds () //disables multiple executions
        ansiColor('xterm')
    }

    stages {
        stage ('Install Dependencies') {
            steps {    
                sh """
                npm install
                """
            }    
        }

    }    
    post { //useful as alert for success or failure

        always {
            echo 'I will always say hello'
            deleteDir()    //to delete workspace after build
        }

        success {
            echo 'I will run when pipeline is success'
        }

        failure {
            echo 'I will run when pipeline is failure'
            // we configure with slack, when failed we get messege
        }
    }
}