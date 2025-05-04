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
        
        // Run Juice Shop container
        sh '''
            docker run --name juice-shop -d --rm \
                -p 3000:3000 \
                bkimminich/juice-shop
            sleep 5
        '''
        
        // Copy ZAP configuration folder from the workspace to the container
        sh '''
            docker cp ${WORKSPACE}/zap zap:/zap/wrk/zap
        '''
        
        // Run ZAP with the passive scan configuration
        sh '''
            docker run --name zap \
                --add-host=host.docker.internal:host-gateway \
                -v ${WORKSPACE}/results:/zap/wrk/results \
                -w /zap/wrk \
                -t ghcr.io/zaproxy/zaproxy:stable bash -c \
                "zap.sh -cmd -addonupdate; \
                zap.sh -cmd -addoninstall communityScripts; \
                zap.sh -cmd -addoninstall pscanrulesAlpha; \
                zap.sh -cmd -addoninstall pscanrulesBeta; \
                zap.sh -cmd -autorun /zap/wrk/zap/passive_scan.yaml" || true
        '''
    }
    post {
        always {
            // Copy the report files from the container to the Jenkins workspace
            sh '''
                docker cp zap:/zap/wrk/reports/zap_html_report.html ${WORKSPACE}/results/zap_html_report.html || true
                docker cp zap:/zap/wrk/reports/zap_xml_report.xml ${WORKSPACE}/results/zap_xml_report.xml || true
                docker stop zap juice-shop || true
                docker rm zap juice-shop || true
            '''
        }
    }
}
    }
}
