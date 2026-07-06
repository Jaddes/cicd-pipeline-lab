pipeline {
  agent any

  tools {
    nodejs 'Node 7.8.0'
  }

  environment {
    IMAGE_VERSION = 'v1.0'
  }

  stages {
    stage('checkout') {
      steps {
        checkout scm
      }
    }

    stage('prepare env') {
      steps {
        script {
          env.DEPLOY_ENV = env.BRANCH_NAME == 'dev' ? 'dev' : 'main'
          env.APP_PORT = env.DEPLOY_ENV == 'dev' ? '3001' : '3000'
          env.IMAGE_NAME = "node${env.DEPLOY_ENV}:${env.IMAGE_VERSION}"
          env.CONTAINER_NAME = "node-${env.DEPLOY_ENV}"
        }
        echo "Deploying ${env.DEPLOY_ENV} to port ${env.APP_PORT}"
      }
    }

    stage('build') {
      steps {
        script {
          if (isUnix()) {
            sh 'npm install'
            sh 'npm run build'
          } else {
            bat 'npm install'
            bat 'npm run build'
          }
        }
      }
    }

    stage('test') {
      steps {
        script {
          if (isUnix()) {
            sh 'CI=true npm test -- --watchAll=false'
          } else {
            bat 'set CI=true&& npm test -- --watchAll=false'
          }
        }
      }
    }

    stage('build docker image') {
      steps {
        script {
          if (isUnix()) {
            sh 'docker build -t "${IMAGE_NAME}" .'
          } else {
            bat 'docker build -t "%IMAGE_NAME%" .'
          }
        }
      }
    }

    stage('deploy') {
      steps {
        script {
          if (isUnix()) {
            sh '''
              docker rm -f "${CONTAINER_NAME}" || true
              docker run -d --name "${CONTAINER_NAME}" -p "${APP_PORT}:3000" "${IMAGE_NAME}"
            '''
          } else {
            bat '''
              docker rm -f "%CONTAINER_NAME%" 2>NUL
              docker run -d --name "%CONTAINER_NAME%" -p "%APP_PORT%:3000" "%IMAGE_NAME%"
            '''
          }
        }
      }
    }
  }
}
