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
            echo 'Cloning the Git repository...'
            git credentialsId: 'GitHub', url: "https://github.com/${ORGANIZATION_NAME}/${SERVICE_NAME}.git"
            echo 'Git repository cloned.'
         }
      }

      stage('Setup Maven Repository') {
         steps {
            echo 'Setting up Maven repository directory...'
            sh 'mkdir -p $WORKSPACE/.m2/repository'  // Ensure the repository directory exists
            sh 'chmod -R 777 $WORKSPACE/.m2'  // Set permissions for the Maven repository
            echo 'Maven repository setup completed.'
         }
      }

      stage('Build') {
         steps {
            echo 'Verifying Maven version...'
            sh 'mvn -version'  // Verify Maven version
            echo 'Building the project...'
            sh 'mvn clean package'  // Run Maven build
            echo 'Build completed successfully.'
         }
      }

      stage('Build and Push Docker Image') {
         steps {
            script {
                echo "Building Docker image with tag ${REPOSITORY_TAG}..."
                sh "docker image build -t ${REPOSITORY_TAG} ."
                echo "Docker image built..."
            }
         }
      }

      stage('Deploy to Cluster') {
         steps {
            script {
                echo 'Deploying to Kubernetes cluster...'
                // Using docker image with explicit kubectl command
                docker.image('lachlanevenson/k8s-kubectl:latest').inside("--entrypoint=''") {
                    echo 'Running kubectl with envsubst for YAML substitution'
                    sh 'envsubst < ${WORKSPACE}/deploy.yaml | kubectl apply -f -'  // Apply Kubernetes configuration
                }
                echo 'Deployment to Kubernetes cluster completed.'
            }
         }
      }
   }

   post {
      success {
         echo 'Build and deployment successful!'
      }
      failure {
         echo 'Build or deployment failed. Please check the logs for errors.'
      }
   }
}
