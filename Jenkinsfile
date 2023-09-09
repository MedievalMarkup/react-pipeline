pipeline {

  environment {
    dockerimagename = "adil22/react-app:${env.BUILD_NUMBER}"
    dockerImage = ""
    PAT = credentials('gitlab-token')
    KUBECONFIG = "/var/lib/jenkins/.kube/config"
  }

  agent any

  stages {

    stage('Checkout Source') {
      steps {
        script {
          // Bind the GitHub credentials using withCredentials block - same for github
            withCredentials([usernamePassword(credentialsId: 'gitlab', usernameVariable: 'GITLAB_USERNAME', passwordVariable: 'GITLAB_TOKEN')]) {
              git credentialsId: 'gitlab', branch: 'main', url: 'http://$PAT@gitlab.example.com:8090/root/react-pipeline.git'
            }
          }
        } 
      } 
    

    stage('Build image') {
      steps{
        script {
          dockerImage = docker.build dockerimagename
        }
      }
    }

    stage('Pushing Image') {

      environment {
        registryCredential = 'DockerHubPwd'
      }

      steps{
        script {
          docker.withRegistry( 'https://registry.hub.docker.com', registryCredential ) {
            dockerImage.push("${env.BUILD_NUMBER}")
          }
        }
      }
    }

    stage ('Deploy on kubernetes') {

      steps {
        script{
          // def image_id = dockerImage
          sh "ansible-playbook -vvv playbookMK8s.yaml --extra-vars \"image_id=${dockerimagename}\""
        }
      }
    }
  }
} 
