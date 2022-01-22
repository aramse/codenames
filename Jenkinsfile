import static io.aramse.f8.Utils.*

def MAIN_BRANCHES
def MERGED_BRANCH

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

    BUILD_CMD = "f8 build --push"
    DEPLOY_CMD = "f8 deploy"
    TEST_CMD = "f8 test"
  }

  stages {
    stage('Setup') {
      steps {
        script {
          MAIN_BRANCHES = ["master", "main"]
          MERGED_BRANCH = getMergedBranch(this)
        }
      }
    }
    stage('Delete Merged Env') {
      when {
        expression {
          MAIN_BRANCHES.contains(env.BRANCH_NAME) && MERGED_BRANCH
        }
      }
      steps {
        sh 'f8 delete --env ' + MERGED_BRANCH
      }
    }
    stage('Build') {
      steps {
        script {
          if (env.REG_USER != "") { 
            docker.withRegistry(env.REG_AUTH_URL, env.REG_CREDS_ID) {
              sh env.BUILD_CMD
            }
          } else {
            sh env.BUILD_CMD
          }
        }
      }
    }
    stage('Deploy') {
      steps {
        sh env.DEPLOY_CMD
      }
    }
    stage('Test') {
      steps {
        sh env.TEST_CMD
      }
    }
  }
  post {
    success {
      script {
        if (env.SLACK_ENABLED == 'true') {
          slackNotify(this, 'SUCCESS')
        }
      }
    }
    failure {
      script {
        if (env.SLACK_ENABLED == 'true') {
          slackNotify(this, 'FAILURE')
        }
      }
    }
  }
}
