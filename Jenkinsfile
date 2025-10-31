pipeline {
  agent {
    label 'nav-ubuntu-a'
  }
  environment {
    APP_NAME = "flask-counter"
  }
  stages {
    stage('Build Docker Image') {
      steps {
        sh '''
          cd app
          docker rm -f ${APP_NAME} || true
          docker image rmi -f ${APP_NAME}:latest || true
          docker build -t ${APP_NAME}:latest .
        '''
      }
    }
    /*stage('Run Monitoring Stack') {
      steps {
        sh '''
          docker compose -f docker-compose.yml up -d
          sleep 10
        '''
      }
    }*/
    stage('Run App Container') {
      steps {
        sh '''

          docker run -d --name ${APP_NAME} --network prom-grafana_monitoring -p 8010:5000 ${APP_NAME}:latest
        '''
      }
    }
    stage('Validate Metrics') {
      steps {
        sh '''
          echo "Prometheus Targets:"
          curl -s http://44.200.35.14:8006/api/v1/targets | jq '.data.activeTargets'
        '''
      }
    }
    stage('Health Check') {
      steps {
        script {
          def resp = sh(script: "curl -s -o /dev/null -w '%{http_code}' http://44.200.35.14:8010", returnStdout: true).trim()
          if (resp != '200') {
            error "Application failed health check!"
          } else {
            echo "App is healthy ✅"
          }
        }
      }
    }
  }
  post {
    always {
      sh 'docker ps'
    }
  }
}