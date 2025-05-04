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
        // Tworzymy katalog na wyniki
        sh 'mkdir -p results/'

        // Uruchamiamy kontener Juice Shop
        sh '''
            docker run --name juice-shop -d --rm \
                -p 3000:3000 \
                bkimminich/juice-shop
            sleep 5
        '''

        // Sprawdzamy pliki w lokalnym katalogu 'zap' w Jenkins
        sh 'ls -la ${WORKSPACE}/zap'

        // Sprawdzamy, czy plik jest dostępny w kontenerze ZAP
        sh '''
            docker run --rm \
                --add-host=host.docker.internal:host-gateway \
                -v ${WORKSPACE}/zap:/zap/wrk \
                -t ghcr.io/zaproxy/zaproxy:stable bash -c "
                    echo '=== Listing files in /zap/wrk ===';
                    ls -la /zap/wrk;
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
            // Kopiowanie raportów z kontenera ZAP do Jenkins
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
