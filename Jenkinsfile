pipeline {
  agent any
  tools { nodejs 'NodeJS 18' }   // pastikan nama tool sama dengan yang kamu set di Manage Jenkins > Tools
  options { timestamps() }
  environment { CI = 'true' }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
        sh 'echo "Branch: $(git rev-parse --abbrev-ref HEAD)" 2>&1 | tee -a pipeline.log'
      }
    }

    stage('Install') {
      steps {
        sh 'echo "== npm ci ==" 2>&1 | tee -a pipeline.log'
        sh 'npm ci 2>&1 | tee -a pipeline.log'
      }
    }

    stage('Build') {
      steps {
        sh 'echo "== npm run build ==" 2>&1 | tee -a pipeline.log'
        sh 'npm run build 2>&1 | tee -a pipeline.log'
      }
    }

    stage('Test') {
      steps {
        sh 'echo "== npm test -- --watchAll=false ==" 2>&1 | tee -a pipeline.log'
        sh 'npm test -- --watchAll=false 2>&1 | tee -a pipeline.log'
      }
    }

    stage('Archive build') {
      when { expression { fileExists("build") } }
      steps {
        archiveArtifacts artifacts: 'build/**', fingerprint: true
      }
    }
  }

  post {
    always {
      sh 'cp -f pipeline.log log.txt || true'
      archiveArtifacts artifacts: 'pipeline.log,log.txt', allowEmptyArchive: true
    }
  }
}
