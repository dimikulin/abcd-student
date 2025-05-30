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
                        docker stop zap || true
                        docker rm zap || true
                        docker stop juice-shop || true
                    '''
                    archiveArtifacts artifacts: 'results/*.html, results/*.xml', allowEmptyArchive: true
                }
            }
        }

        stage('OSV-Scanner Analysis') {
            steps {
                sh 'mkdir -p results/'

                // Pobierz OSV-Scanner jeśli nie jest zainstalowany
                sh '''
                    if ! command -v osv-scanner &> /dev/null; then
                        curl -sSL https://github.com/google/osv-scanner/releases/download/v1.0.0/osv-scanner-linux-amd64 -o /usr/local/bin/osv-scanner
                        chmod +x /usr/local/bin/osv-scanner
                    fi
                '''

                // Uruchom skanowanie
                sh '''
                    if [ -f package-lock.json ]; then
                          osv-scanner scan --lockfile package-lock.json > results/osv_scan_report.json || true
                    else
                        echo "package-lock.json not found. Skipping OSV scan."
                    fi
                '''
            }

            post {
                always {
                    archiveArtifacts artifacts: 'results/osv_scan_report.json', allowEmptyArchive: true
                }
            }
        }
        
         stage('TruffleHog Scan') {
            steps {
                sh 'mkdir -p results/'
                sh '''
                   trufflehog git file://. --only-verified --json > results/trufflehog_report.json || true
                '''
            }

            post {
                always {
                    archiveArtifacts artifacts: 'results/trufflehog_report.json', allowEmptyArchive: true
                }
            }
        }

        stage('Semgrep Analysis') {
            steps {
                sh 'mkdir -p results/'

                // Instalacja Semgrep
                sh '''
                    if ! command -v semgrep &> /dev/null; then
                        echo "Semgrep not found, installing..."
                        pip install semgrep
                    else
                        echo "Semgrep already installed."
                    fi
                '''

                // Wykonanie skanowania
                sh '''
                    semgrep --config auto --json > results/semgrep_report.json || true
                '''
            }

            post {
                always {
                    archiveArtifacts artifacts: 'results/semgrep_report.json', allowEmptyArchive: true
                }
            }
        }
    }
}
