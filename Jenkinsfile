@Library('adda213-share-library')_
pipeline {
     environment {
       DOCKERHUB_ID = "adda213"
       IMAGE_NAME = "ic-webapp" 
       IMAGE_TAG = "latest"
       DOCKERHUB_PASSWORD = credentials('dockerhub_password')
     }
     agent none
     stages {
         stage('Build image') {
             agent any
             steps {
                script {
                  sh 'docker build --no-cache -f ./ic-webapp/Dockerfile -t ${DOCKERHUB_ID}/$IMAGE_NAME:$IMAGE_TAG .'  
                }
             }
         }
         stage('run container based on build image') {
             agent any
             steps {
                script {
                     sh '''
                     docker ps -a | grep -i $IMAGE_NAME && docker rm -f ${IMAGE_NAME}
                     docker run --name $IMAGE_NAME -d -p 80:8080 ${DOCKERHUB_ID}/$IMAGE_NAME:$IMAGE_TAG
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
                     curl -I http://192.168.56.16 | grep -i "200"
                  '''  
                }
             }
         }
         stage('Clean Container') {
             agent any
             steps {
                script {
                  sh '''
                     docker stop $IMAGE_NAME
                     docker rm -f $IMAGE_NAME
                  '''  
                }
             }
         }
         stage('push image in staging and deploy it') {
             agent any
             steps {   script {
                  sh '''
                     echo $DOCKERHUB_PASSWORD | docker login -u $DOCKERHUB_ID --password-stdin
                     docker push ${DOCKERHUB_ID}/$IMAGE_NAME:$IMAGE_TAG
                  '''  
                }
             }

         }
        
     }
}
