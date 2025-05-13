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

        stage('TruffleHog Scan') {
            steps {
                sh 'mkdir -p results/'
                sh '''
                    trufflehog git file://. --since-commit main --branch feature/example --only-verified --fail > results/trufflehog_report.txt || true
                '''
            }

            post {
                always {
                    archiveArtifacts artifacts: 'results/trufflehog_report.txt', allowEmptyArchive: true
                }
            }
        }
    }
}
