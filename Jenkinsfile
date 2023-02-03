/* import shared library */
@Library('shared-library')_

pipeline {
    environment {
        IMAGE_NAME = "alpinehelloworld"
        APP_EXPOSED_PORT = "80"
        IMAGE_TAG = "latest"
        STAGING = "dajaj-staging"
        PRODUCTION = "dajaj-prod"
        DOCKERHUB_ID = "dajaj"
        DOCKERHUB_PASSWORD = credentials('DockerHub')
        APP_NAME = "dajaj"
        STG_API_ENDPOINT = "34.203.201.141:1993"
        STG_APP_ENDPOINT = "34.203.201.141:8000"
        PROD_API_ENDPOINT = "34.203.201.141:1993"
        PROD_APP_ENDPOINT = "34.203.201.141"
        INTERNAL_PORT = "5000"
        EXTERNAL_PORT = "${PORT_EXPOSED}"
        CONTAINER_IMAGE = "${DOCKERHUB_ID}/${IMAGE_NAME}:${IMAGE_TAG}"
    }
    agent none
    stages {
       stage('Build image') {
           agent any
           steps {
              script {
                sh 'docker build -t ${CONTAINER_IMAGE} .'
              }
           }
       }
       stage('Run container based on builded image') {
          agent any
          steps {
            script {
              sh '''
                docker rm -f ${IMAGE_NAME}
                docker run -d -p 8085:5000 -e PORT=5000 --name ${IMAGE_NAME} ${CONTAINER_IMAGE} 
                sleep 5
              '''
             }
          }
       }
       stage('Test image') {
           agent any
           steps {
              script {
                sh '''
                   curl http://172.17.0.1:8085 | grep -q "Hello world!"
                '''
              }
           }
       }
       stage('Clean container') {
          agent any
          steps {
             script {
               sh '''
                  docker stop ${IMAGE_NAME}
                  docker rm  ${IMAGE_NAME}
               '''
             }
          }
      }

      stage ('Login and Push Image on docker hub') {
          agent any
          steps {
             script {
               sh '''
                  echo $DOCKERHUB_PASSWORD_PSW | docker login -u $DOCKERHUB_PASSWORD_USR --password-stdin
                  docker push ${CONTAINER_IMAGE}
               '''
             }
          }
      }

      stage('STAGING - Deploy app') {
      agent any
      steps {
          script {
            sh """
              echo  {\\"your_name\\":\\"${APP_NAME}\\",\\"container_image\\":\\"${CONTAINER_IMAGE}\\", \\"external_port\\":\\"${EXTERNAL_PORT}00\\", \\"internal_port\\":\\"${INTERNAL_PORT}\\"}  > data.json 
              curl -v -X POST http://${STG_API_ENDPOINT}/staging -H 'Content-Type: application/json'  --data-binary @data.json  2>&1 | grep 200
            """
          }
        }
     
     }
     stage('PROD - Deploy app') {
       when {
           expression { GIT_BRANCH == 'origin/master' }
       }
     agent any

       steps {
          script {
            sh """
              echo  {\\"your_name\\":\\"${APP_NAME}\\",\\"container_image\\":\\"${CONTAINER_IMAGE}\\", \\"external_port\\":\\"${EXTERNAL_PORT}\\", \\"internal_port\\":\\"${INTERNAL_PORT}\\"}  > data.json 
              curl -v -X POST http://${PROD_API_ENDPOINT}/prod -H 'Content-Type: application/json'  --data-binary @data.json  2>&1 | grep 200
            """
          }
       }
     }
  }
  post {
    always {
      script {
        slackNotifier currentBuild.result
      }
    }  
  }
}