pipeline {
  agent any
  options {
    disableConcurrentBuilds()
  }  
  environment {
    COMMIT_HASH = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
    F8_TAG = "$BRANCH_NAME-$BUILD_ID-$COMMIT_HASH"
    F8_ENVIRONMENT = "$BRANCH_NAME"
    F8_ENV_TYPE = "dev"
    F8_LOG_LINKS = "true"
  }

  stages {
    stage('Build') {
      steps {
        withDockerRegistry([url: 'https://gcr.io', credentialsId: 'reg-creds']) {
          sh 'f8 build --push'
        }
      }
    }
    stage('Deploy') {
      steps {
        sh 'f8 deploy'
      }
    }
    stage('Test') {
      steps {
        sh 'f8 test'
      }
    }
  }
}
