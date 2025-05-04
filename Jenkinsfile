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

        stage('Verify zap config') {
            steps {
                sh 'ls -la zap/'
            }
        }

        stage('Debug ZAP config visibility inside container') {
            steps {
                sh '''
                    echo "Files visible INSIDE ZAP container:"
                    docker run --rm \
                        --add-host=host.docker.internal:host-gateway \
                        -v ${WORKSPACE}/zap:/zap/wrk/:rw \
                        -w /zap/wrk \
                        -t ghcr.io/zaproxy/zaproxy:stable bash -c "ls -la /zap/wrk"
                '''
            }
        }

      
    }
}
