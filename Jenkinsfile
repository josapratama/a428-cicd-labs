pipeline {
  agent any
  tools { nodejs 'NodeJS 18' }
  options { timestamps() }
  environment { CI = 'true' }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
        sh 'echo "Branch: $(git rev-parse --abbrev-ref HEAD)" |& tee -a pipeline.log'
      }
    }
    stage('Install') {
      steps { sh 'npm install |& tee -a pipeline.log' }
    }
    stage('Build') {
      steps { sh 'npm run build |& tee -a pipeline.log' }
    }
    stage('Test') {
      steps { sh 'npm test -- --watchAll=false |& tee -a pipeline.log' }
    }
    stage('Archive build') {
      when { expression { fileExists("build") } }
      steps { archiveArtifacts artifacts: 'build/**', fingerprint: true }
    }
  }
  post {
    always {
      sh 'cp -f pipeline.log log.txt || true'
      archiveArtifacts artifacts: 'pipeline.log,log.txt', allowEmptyArchive: true
    }
  }
}
