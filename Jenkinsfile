pipeline {
  agent {
    label 'jenkins-git'
  }

  env
  stages {
    stage('Checkout') {
      steps {
        git branch: 'main',
            url: 'https://github.com/GayanaJinde/devops-learning'
      }
    }
    stage('Build') {
      steps {
        echo 'Build stage'
        sh 'pwd'
        sh 'ls -lrt'
      }
    }
    stage('Test') {
      agent {
        label 'ubuntu'
      }
      steps {
        echo 'Test stage'
      }
    }
  }
}
