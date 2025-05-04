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
        sh 'ls -la ${WORKSPACE}/zap'
        sh """
    docker run --name zap \
        --add-host=host.docker.internal:host-gateway \
        -v "${WORKSPACE}/zap:/zap/wrk:rw" \
        -t ghcr.io/zaproxy/zaproxy:stable bash -c '
            echo "=== Listing files in /zap/wrk ===";
            ls -la /zap/wrk;
            echo "=== Starting ZAP ===";
            zap.sh -cmd -addonupdate;
            zap.sh -cmd -addoninstall communityScripts;
            zap.sh -cmd -addoninstall pscanrulesAlpha;
            zap.sh -cmd -addoninstall pscanrulesBeta;
            zap.sh -cmd -autorun /zap/wrk/passive_scan.yaml
        ' || true
"""
    }
    post {
        always {
            sh '''
                docker cp zap:/zap/wrk/reports/zap_html_report.html ${WORKSPACE}/results/zap_html_report.html
                docker cp zap:/zap/wrk/reports/zap_xml_report.xml ${WORKSPACE}/results/zap_xml_report.xml
                docker stop zap juice-shop
                docker rm zap
            '''
        }
    }
}

    }
}
