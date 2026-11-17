pipeline {
  agent any
  options { timestamps() }

  environment {
    PATH  = "/usr/local/bin:/opt/homebrew/bin:/Applications/Docker.app/Contents/Resources/bin:${env.PATH}"
    IMAGE = 'monapp'
    TAG   = "build-${env.BUILD_NUMBER}"
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Vérification Docker') {
      steps {
        sh 'whoami'
        sh 'docker version'
        sh 'docker compose version'
      }
    }

    stage('Docker Build') {
      steps {
        sh 'docker build -t $IMAGE:$TAG -t $IMAGE:dev .'
      }
    }

    stage('Smoke Test') {
      steps {
        sh '''
          docker rm -f monapp_test 2>/dev/null || true
          docker run -d --name monapp_test -p 8081:80 $IMAGE:$TAG
          sleep 3
          curl -I http://localhost:8081 | grep "200 OK"
          curl -s http://localhost:8081 | grep "projet simple"
          docker rm -f monapp_test
        '''
      }
    }

    stage('Deploy') {
      steps {
        sh 'docker compose -p tp2_jenkins_git up -d --force-recreate'
        sh 'docker ps'
      }
    }
  }

  post {
    success { echo 'Build + Test + Deploy OK' }
    failure { echo 'Build / Tests / Deploy KO' }
  }
}
