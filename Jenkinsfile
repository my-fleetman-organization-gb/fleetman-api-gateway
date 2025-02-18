pipeline {
   agent any

   tools {
        maven 'Maven 3.x'  // Reference the Maven installation you configured in Global Tool Configuration
    }
   
   environment {
     SERVICE_NAME = "fleetman-api-gateway"
     REPOSITORY_TAG = "richardchesterwood/k8s-fleetman-api-gateway:performance"
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
                sh "docker image build -t ${REPOSITORY_TAG} ."  // Build the Docker image
                echo "Docker image built..."

                echo "Logging in to Docker Hub..."
                withDockerRegistry([credentialsId: 'Docker_Credentials', url: 'https://index.docker.io/v1/']) {
                    echo "Pushing Docker image to registry..."
                    sh "docker push ${REPOSITORY_TAG}"  // Push the image to the registry
                    echo "Docker image pushed to registry..."
                }
            }
         }
      }

      stage('Deploy to Cluster') {
         steps {
            script {
                echo 'Deploying to Kubernetes cluster...'
                docker.image('lachlanevenson/k8s-kubectl:latest').inside("--entrypoint=''") {
                    echo 'Running deploy.yaml '
                    sh 'kubectl apply -f deploy.yaml'  // Apply Kubernetes configuration
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
