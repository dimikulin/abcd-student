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
                sh 'ls -la'
            }
        }
      stage('ZAP DAST Scan') {
    steps {
        sh '''
            docker run --rm \
              --network host \
              -v $(pwd):/zap/wrk/:rw \
              ghcr.io/zaproxy/zap-baseline:stable \
              -t http://host.docker.internal:3000 \
              -r zap-report.html \
              -J zap-report.json
        '''
    }
}
post {
    always {
        echo 'Archiwizuję raport ZAP...'
        archiveArtifacts artifacts: 'zap-report.*', fingerprint: true, allowEmptyArchive: true
    }
}
    }
}
