pipeline {
    agent {
        docker {
            image 'node:lts-buster-slim'
            args '-p 3000:3000'
        }
    }
    environment {
        CI = 'true'
    }
    stages {
        stage('Build') {
            steps {
                sh 'echo "===== PIPELINE START =====" > log.txt'
                sh 'echo "Build #${BUILD_NUMBER} - $(date)" >> log.txt'
                sh 'echo "" >> log.txt'
                sh 'echo "--- Stage: Build ---" >> log.txt'
                sh 'npm install 2>&1 | tee -a log.txt'
            }
        }
        stage('Test') {
            steps {
                sh 'echo "" >> log.txt'
                sh 'echo "--- Stage: Test ---" >> log.txt'
                sh './jenkins/scripts/test.sh 2>&1 | tee -a log.txt'
            }
        }
        stage('Manual Approval') {
            steps {
                sh 'echo "" >> log.txt'
                sh 'echo "--- Stage: Manual Approval ---" >> log.txt'
                sh 'echo "Waiting for manual approval..." >> log.txt'
                timeout(time: 15, unit: 'MINUTES') {
                    input message: 'Lanjutkan ke tahap Deploy?', ok: 'Proceed'
                }
                sh 'echo "Manual approval granted." >> log.txt'
            }
        }
        stage('Deploy') {
            steps {
                sh 'echo "" >> log.txt'
                sh 'echo "--- Stage: Deploy ---" >> log.txt'
                sh './jenkins/scripts/deliver.sh 2>&1 | tee -a log.txt'
                sh 'echo "Aplikasi berjalan, menunggu 60 detik..." >> log.txt'
                sleep time: 60, unit: 'SECONDS'
                sh './jenkins/scripts/kill.sh 2>&1 | tee -a log.txt'
                sh 'echo "Aplikasi dihentikan." >> log.txt'
            }
        }
    }
    post {
        success {
            sh 'echo "" >> log.txt'
            sh 'echo "===== PIPELINE SUCCESS =====" >> log.txt'
            sh 'echo "Finished: $(date)" >> log.txt'
        }
        failure {
            sh 'echo "" >> log.txt'
            sh 'echo "===== PIPELINE FAILED =====" >> log.txt'
            sh 'echo "Finished: $(date)" >> log.txt'
        }
        aborted {
            sh 'echo "" >> log.txt'
            sh 'echo "===== PIPELINE ABORTED =====" >> log.txt'
            sh 'echo "Finished: $(date)" >> log.txt'
        }
        always {
            archiveArtifacts artifacts: 'log.txt', allowEmptyArchive: true
        }
    }
}
