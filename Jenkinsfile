pipeline {
  agent any
  tools { nodejs 'NodeJS 18' }   // samakan dengan Manage Jenkins > Tools
  options { timestamps() }
  environment {
    CI = 'true'
    APP_IMAGE = "react-app-ci:${env.BUILD_NUMBER}"
    APP_CONTAINER = "react-app-ci-${env.BUILD_NUMBER}"
    APP_PORT = "3000" // ganti jika perlu
  }

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

    // === Kriteria 4: Manual Approval sebelum Deploy ===
    stage('Manual Approval') {
      steps {
        timeout(time: 15, unit: 'MINUTES') {
          input message: 'Lanjutkan ke tahap Deploy?', ok: 'Proceed'
        }
      }
    }

    // === Kriteria 2 & 3: Deploy + jeda 1 menit, lalu stop ===
    stage('Deploy') {
      steps {
        script {
          // Dockerfile minimal untuk serve build via nginx
          writeFile file: 'Dockerfile.deploy', text: """
          FROM nginx:alpine
          COPY build/ /usr/share/nginx/html
          EXPOSE 80
          """

          sh """
            echo "== Build image deploy =="
            docker build -t "${APP_IMAGE}" -f Dockerfile.deploy .

            echo "== Run container =="
            docker run -d --name "${APP_CONTAINER}" -p ${APP_PORT}:80 "${APP_IMAGE}"
          """

          echo "Aplikasi berjalan selama 60 detik di http://<host>:${APP_PORT}"
          sleep time: 60, unit: 'SECONDS'  // jeda 1 menit sesuai kriteria

          sh """
            echo "== Stop & remove container =="
            docker stop "${APP_CONTAINER}" || true
            docker rm "${APP_CONTAINER}" || true
          """
        }
      }
    }
  }

  post {
    always {
      sh 'cp -f pipeline.log log.txt || true'
      archiveArtifacts artifacts: 'pipeline.log,log.txt,**/npm-*.log,**/yarn-*.log', allowEmptyArchive: true
      cleanWs()
    }
  }
}
