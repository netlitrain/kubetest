pipeline {
  agent any

  options {
   timestamps()
}


  environment {
        IMAGE_NAME = "trainerbpl10/knbimage"
    	  IMAGE_TAG = "${BUILD_NUMBER}"
    	  DOCKER_CREDS = credentials('dockerhub-creds')
    }

  stages {
    stage('CODE') {
      steps {
        git url:"https://github.com/netlitrain/kubetest.git", branch:"main"
      }
    }

    
    stage('BUILD') {
      	    steps {
        	    sh " docker build -t $IMAGE_NAME:$IMAGE_TAG . "
      		}
    	}

    stage("IMAGE_CHECK") {
            steps {

                sh " trivy image --severity CRITICAL --exit-code 0 $IMAGE_NAME:$IMAGE_TAG "
            }
        }
    
    stage('IMAGE_PUSH') {
      steps {
        sh " echo $DOCKER_CREDS_PSW | docker login -u $DOCKER_CREDS_USR --password-stdin "
        sh " docker push $IMAGE_NAME:$IMAGE_TAG "
      }
    }
    
    stage("Deploy to K8s") {
            steps {
                withAWS(credentials: 'aws-creds') {
                sh 'aws eks --region us-east-1 update-kubeconfig --name netlicluster'
                sh "sed -i 's|IMAGE_PLACEHOLDER|${IMAGE_NAME}:${IMAGE_TAG}|g' k8s/deployment.yml"
                        sh "kubectl apply -f k8s/deployment.yml"
                        sh "kubectl apply -f k8s/service.yml"
                }   
            }
        } 
  }
}
