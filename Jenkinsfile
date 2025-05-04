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

        stage('Fix permissions') {
    steps {
        sh 'chmod -R 755 zap/'
    }
}
          stage('Verify zap config 2') {
            steps {
                sh 'ls -la zap/'
            }
        }


stage('[ZAP] Baseline passive-scan') {
    steps {
        sh 'mkdir -p results/'
        sh '''
            docker run --name juice-shop -d --rm -p 3000:3000 bkimminich/juice-shop
            sleep 5
        '''

        // Sprawdzenie katalogu przed uruchomieniem kontenera ZAP
        sh 'echo "Mounting ${WORKSPACE}/zap to /zap/wrk"; ls -la ${WORKSPACE}/zap'
     sh '''
    docker run --rm --add-host=host.docker.internal:host-gateway -v /zap:/zap/wrk -t ghcr.io/zaproxy/zaproxy:stable bash -c "
        echo '=== Listing files in /zap ===';
        ls -la /zap;
        echo '=== Checking passive_scan.yaml content ===';
        cat /zap/wrk/passive_scan.yaml || echo 'MISSING passive_scan.yaml';
        echo '=== Starting ZAP ===';
        zap.sh -cmd -addonupdate;
        zap.sh -cmd -addoninstall communityScripts;
        zap.sh -cmd -addoninstall pscanrulesAlpha;
        zap.sh -cmd -addoninstall pscanrulesBeta;
        zap.sh -cmd -autorun /zap/wrk/passive_scan.yaml
    "
'''
    }
    post {
        always {
            // Sprawdzenie, czy kontener ZAP działa przed próbą kopiowania raportów
            sh '''
                docker ps -a
                docker cp zap:/zap/wrk/reports/zap_html_report.html ${WORKSPACE}/results/zap_html_report.html
                docker cp zap:/zap/wrk/reports/zap_xml_report.xml ${WORKSPACE}/results/zap_xml_report.xml
                docker stop zap
                docker rm zap
            '''
        }
    }
}

    }
}
