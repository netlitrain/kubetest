pipeline {
  agent any

  options {
   timestamps()
   buildDiscarder (logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '1', numToKeepStr: '2'))
}


  environment {
    IMAGE_NAME = "webimg"
    IMAGE_TAG = "${BUILD_NUMBER}"
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

    stage('IMAGE_CHECK') {
      steps {
        sh " trivy image --severity CRITICAL --exit-code 0 $IMAGE_NAME:$IMAGE_TAG "
      }
    }

    stage('AWS Deploy') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'awscreds'
                ]]) {
                    sh '''
                        aws eks --region us-east-1 update-kubeconfig --name netlicluster
                        kubectl get pods
                        kubectl run netli --image=$IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }
  }
}
