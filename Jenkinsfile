pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
    }

    stages {
        stage('Code checkout from GitHub') {
            steps {
                script {
                    cleanWs()
                    git credentialsId: 'github-pat', url: 'https://github.com/dimikulin/abcd-student.git', branch: 'main'
                }
            }
        }

        stage('Example') {
            steps {
                echo 'Hello!'
                sh 'docker ps'
                sh 'curl http://host.docker.internal:3000'
            }
        }
    }
}
