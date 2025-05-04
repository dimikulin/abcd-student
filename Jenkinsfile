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

        stage('[ZAP] Baseline passive-scan') {
    steps {
        sh 'mkdir -p results/'
       sh '''
    docker run --rm --name zap \
        --add-host=host.docker.internal:host-gateway \
        -v /var/jenkins_home/workspace/Example/zap:/zap/wrk/:rw \
        -t ghcr.io/zaproxy/zaproxy:stable bash -c \
        "zap.sh -cmd -addonupdate; \
         zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta; \
         zap.sh -cmd -autorun /zap/passive_scan.yaml" \
        || true
'''
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
