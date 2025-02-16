pipeline {
   agent any

   tools {
        maven 'Maven 3.x'  // Reference the Maven installation you configured in Global Tool Configuration
    }
   
   environment {
     SERVICE_NAME = "fleetman-api-gateway"
     REPOSITORY_TAG="${YOUR_DOCKERHUB_USERNAME}/${ORGANIZATION_NAME}-${SERVICE_NAME}:${BUILD_ID}"
     MAVEN_OPTS = '-Dmaven.repo.local=$WORKSPACE/.m2/repository'  // Use a custom local repository in the workspace
   }

   stages {
      stage('Preparation') {
         steps {
            cleanWs()  // Clean workspace before each build
            git credentialsId: 'GitHub', url: "https://github.com/${ORGANIZATION_NAME}/${SERVICE_NAME}.git"
         }
      }

      stage('Setup Maven Repository') {
         steps {
            sh 'mkdir -p $WORKSPACE/.m2/repository'  // Ensure the repository directory exists
            sh 'chmod -R 777 $WORKSPACE/.m2'  // Set permissions for the Maven repository
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
            script {
               // Use a Docker container with kubectl and envsubst installed
               docker.image('lachlanevenson/k8s-kubectl:latest').inside {
                  sh 'envsubst < ${WORKSPACE}/deploy.yaml | kubectl apply -f -'
               }
            }
         }
      }
   }
}
