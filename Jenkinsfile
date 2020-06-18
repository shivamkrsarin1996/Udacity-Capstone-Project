pipeline {
  environment {
    registry = "deepakmadisetty/capstone"
    registryCredential = 'dockerhub'
  }

  agent any 
  
  stages {
    stage('Check Environment') {
        steps {
          echo 'Check Prerequisites'
          sh 'docker -v'
        }
    }

    stage('Linting Files') {
        steps {
            sh 'tidy -q -e *.html'
            sh 'docker run --rm -i hadolint/hadolint < Dockerfile'
        }
    }

    stage('Build, Deploy & Push Docker Image') {
        steps {
          script{
            dockerImage = docker.build registry
            docker.withRegistry( '', registryCredential ) {
              dockerImage.push()
            }
          }
          
        }
    }

    stage('Deploying the app to Kubernetes Cluster') {
        steps {
          echo 'Creating Kubernetes Cluster'
          withAWS(region:'us-west-2',credentials:'awscreds') {
            dir('./') {
              sh '''
              aws eks --region us-west-2 update-kubeconfig --name capstone-eks-cluster
              echo 'Kubernetes Deployment'
              kubectl apply -f kubernetes/config/eks-auth-cm.yml
              kubectl apply -f kubernetes/config/eks-deployment.yml
              kubectl apply -f kubernetes/config/eks-service.yml
              kubectl get nodes
              kubectl get pods
              kubectl get svc service-capstone -o yaml
              '''
            }
          }
        }
    }

    stage('Clean Up') {
        steps {
          sh 'docker system prune'
        }
    }
  }
}