pipeline {
  agent {
    // Jalankan job di dalam container Node LTS
    docker {
      image 'node:lts-buster-slim'
      // Ganti 3001 jika "port is already allocated"
      args '-p 3001:3000 -u root:root'
    }
  }

  options {
    timestamps()
    ansiColor('xterm')
  }

  environment {
    CI = 'true'
    // Sesuaikan kalau paket butuh proxy/registry mirror
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
        sh 'echo "Branch aktif: $(git rev-parse --abbrev-ref HEAD)" |& tee -a pipeline.log'
      }
    }

    stage('Install') {
      steps {
        sh '''
          echo "== npm ci ==" |& tee -a pipeline.log
          npm ci |& tee -a pipeline.log
        '''
      }
    }

    stage('Build') {
      steps {
        sh '''
          echo "== npm run build ==" |& tee -a pipeline.log
          npm run build |& tee -a pipeline.log
        '''
      }
    }

    stage('Test') {
      steps {
        // Non-interaktif, cocok untuk CI
        sh '''
          echo "== npm test ==" |& tee -a pipeline.log
          npm test -- --watchAll=false |& tee -a pipeline.log
        '''
      }
    }

    stage('Archive build') {
      when { expression { fileExists('build') } }
      steps {
        archiveArtifacts artifacts: 'build/**', fingerprint: true
      }
    }
  }

  post {
    always {
      // Pastikan ada log.txt di Artifacts (diminta pada submission)
      sh 'cp -f pipeline.log log.txt || true'
      archiveArtifacts artifacts: 'pipeline.log,log.txt', allowEmptyArchive: true
    }
    success {
      echo 'Pipeline sukses ✅'
    }
    failure {
      echo 'Pipeline gagal ❌ — cek Artifacts: pipeline.log/log.txt'
    }
  }
}
