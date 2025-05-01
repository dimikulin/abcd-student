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
         stage('OWASP ZAP Baseline Scan') {
            steps {
                script {
                    // Uruchomienie ZAP-a i wykonanie pasywnego skanu
                    sh '''
                        docker run --rm -u zap \
                          -v $WORKSPACE:/zap/wrk \
                          owasp/zap2docker-stable \
                          zap-baseline.py -t http://localhost:3000/#/ -r zap-report.html || true
                    '''
                }
            }
        }
        stage('Archive Report') {
            steps {
                // Archiwizuj raport jako artefakt
                archiveArtifacts artifacts: 'zap-report.html', allowEmptyArchive: true
            }
        }
    }

    }
}
