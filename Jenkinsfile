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


      stage('[ZAP] Baseline passive-scan') {
    steps {
        sh 'mkdir -p results/'

        sh '''
            docker run --name juice-shop -d --rm \
                -p 3000:3000 \
                bkimminich/juice-shop
            sleep 5
        '''

        // Start ZAP container in background
        sh '''
            docker run -d --name zap \
                --add-host=host.docker.internal:host-gateway \
                -v ${WORKSPACE}/results:/zap/wrk/results \
                -w /zap/wrk \
                -t ghcr.io/zaproxy/zaproxy:stable sleep 30
        '''

        // Copy zap config folder into container
        sh 'docker cp ${WORKSPACE}/zap zap:/zap/wrk/zap'

        // Run scan using copied config
        sh '''
            docker exec zap bash -c "\
                zap.sh -cmd -addonupdate && \
                zap.sh -cmd -addoninstall communityScripts && \
                zap.sh -cmd -addoninstall pscanrulesAlpha && \
                zap.sh -cmd -addoninstall pscanrulesBeta && \
                zap.sh -cmd -autorun zap/passive_scan.yaml"
        '''
    }
    post {
        always {
            sh '''
                docker cp zap:/zap/wrk/reports/zap_html_report.html ${WORKSPACE}/results/zap_html_report.html || true
                docker cp zap:/zap/wrk/reports/zap_xml_report.xml ${WORKSPACE}/results/zap_xml_report.xml || true
                docker stop zap || true
                docker rm zap || true
            '''
        }
    }
}
    }
}
