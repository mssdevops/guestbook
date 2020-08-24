//This line indicates the Application display name
currentBuild.displayName="MiracleGuestBook-#"+currentBuild.number

//This is the Declarative Pipeline Script
pipeline{
    agent any 
     stages{
//Build the Docker Image based on the Dockerfile
        stage('Build Docker Image'){
	  steps{
	     sh "sudo docker build -t maniengg/php-redis:${BUILD_ID} php-redis/"   //when we run docker in this step, we're running it via a shell on the docker build-pod container
             sh "sudo docker build -t maniengg/redis-follower:${BUILD_ID} redis-follower/"
           }
       }
 // Pushing the Docker Image to DockerHub   
     stage('Push Docker Image'){
	steps{
//Docker Hub Credentials
        withCredentials([string(credentialsId: 'DOKCER_HUB_PASSWORD', variable: 'DOKCER_HUB_PASSWORD')]) {
          sh "sudo docker login -u maniengg -p ${DOKCER_HUB_PASSWORD}"
          sh "sudo docker push maniengg/php-redis:${BUILD_ID}"
	  sh "sudo docker push maniengg/redis-follower:${BUILD_ID}"
            }
        }
     }     
// Application Deploying into K8s Cluster
     stage("Deploy To Kuberates Cluster"){
      steps{
//Created Service Account in GCP console and generated key is added it to the Jenkins Credentials.
        withCredentials([file(credentialsId: 'demo-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
//activate the service account
         sh "gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}"
//Configuring the project details to Jenkins and communicate with the gke cluster
         sh "gcloud config set project mssdevops-284216" //configuring name of the project
         sh "gcloud config set compute/zone us-central1-c"// confiuring the project compute-zone
         sh "gcloud config set compute/region us-central1"// confiuring the project compute-region
         sh "gcloud container clusters get-credentials cluster-1 --zone us-central1-c --project mssdevops-284216"// command to connect the GKE cluster through command line

         sh "sed -i -e 's,image_to_be_deployed,'maniengg/php-redis:${BUILD_ID}',g' frontend-deployment.yaml" // To replace the latest image and reflected in deployment yaml files.
//Kubernetes Deployments and Services
         sh "kubectl apply -f frontend-deployment.yaml"    // This line is to deloy frontend in GKE cluster where it manage a set of identical pods. 
         sh "kubectl apply -f frontend-service.yaml"       //This line is to allocate network access to set of pods.
	 sh "sed -i -e 's,image_to_be_deployed,'maniengg/redis-follower:${BUILD_ID}',g' redis-follower-deployment.yaml" // To replace the latest image and reflected in deployment yaml files.
	 sh "kubectl apply -f redis-follower-deployment.yaml" // This line is to deploy the redis-follower to GKE cluster, where reads data from multiple Redis-follower instances
         sh "kubectl apply -f redis-follower-service.yaml"   // This line is to allocates the netwok access to pods deployed by redis-follower.
	 sh "kubectl apply -f redis-leader-deployment.yaml" // This line is to deploy the redis-leader to GKE cluster,  where the application writes its data to a Redis-leader instance.
	 sh "kubectl apply -f redis-leader-service.yaml"   //  This line is to allocates the netwok access to pods deployed by redis-leader also describes the service resources for redis-leader.
             }
         }
      }
 }
// Email Notification
	post {
        failure {
            script {
                currentBuild.result = 'FAILURE'
            }
        }

        always {
           step([$class: 'Mailer',notifyEveryUnstableBuild: true,recipients: "gjilludimudi@miraclesoft.com,pkannepalli@miraclesoft.com,sarikatla@miraclesoft.com,sakapelly@miraclesoft.com,mkarnam@miraclesoft.com",sendToIndividuals: true])
	
        }
    }
}
