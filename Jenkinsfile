pipeline {
   agent {
      docker {
         image 'maven:3.8.8'
         args '-v /var/jenkins_home/workspace:/var/jenkins_home/workspace'  // Mount workspace
      }
   }

   environment {
     SERVICE_NAME = "fleetman-api-gateway"
     REPOSITORY_TAG="${YOUR_DOCKERHUB_USERNAME}/${ORGANIZATION_NAME}-${SERVICE_NAME}:${BUILD_ID}"
   }

   stages {
      stage('Preparation') {
         steps {
            cleanWs()
            git credentialsId: 'GitHub', url: "https://github.com/${ORGANIZATION_NAME}/${SERVICE_NAME}.git"
         }
      }

      stage('Build') {
         steps {
            sh 'mvn -version'  // Verify Maven version
            sh 'mvn clean package'  // Run Maven build
         }
      }

      stage('Build and Push Image') {
         steps {
           sh 'docker image build -t ${REPOSITORY_TAG} .'
         }
      }

      stage('Deploy to Cluster') {
          steps {
             sh 'envsubst < ${WORKSPACE}/deploy.yaml | kubectl apply -f -'
          }
      }
   }
}
