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

                // Uruchom aplikację Juice Shop
                sh '''
                    docker run --name juice-shop -d --rm -p 3000:3000 bkimminich/juice-shop
                    sleep 5
                '''

                // Uruchom kontener ZAP w tle
                sh '''
                    docker run -d --name zap ghcr.io/zaproxy/zaproxy:stable sleep 1000
                '''

                // UTWÓRZ wymagane foldery wewnątrz kontenera ZAP
                sh '''
                    docker exec zap mkdir -p /zap/wrk/reports
                '''

                // Skopiuj plik konfiguracji do kontenera
                sh 'docker cp zap/passive_scan.yaml zap:/zap/wrk/passive_scan.yaml'

                // Wykonaj skan
                sh '''
                    docker exec zap bash -c "
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
