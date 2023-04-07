pipeline {
  agent any
  environment {
    DOCKERHUB_USERNAME = "mittal0706"
    APP_NAME = "flask_app"
    REGISTRY_CREDS = "dockerhub"
  }
  parameters {
    choice(
      choices: ['dev', 'UAT', 'prod'],
      description: 'Select the branch to build',
      name: 'BRANCH_NAME'
    )
  }
  stages {
    stage('Git Checkout') {
      steps {
        script {
          withCredentials([gitUsernamePassword(credentialsId: 'github', gitToolName: 'Default')]) {
            git branch: "${params.BRANCH_NAME}", credentialsId: 'github', url: 'https://github.com/mittal0706/build_with_parameter.git'
          }
        }
      }
    }
    stage('Install dependencies') {
      steps {
        sh 'pip install -r requirements.txt'
      }
    }
    stage('Docker build image') {
      steps {
        script {
          def imageTag = "${params.BRANCH_NAME}_${BUILD_NUMBER}"
          def imageName = "${DOCKERHUB_USERNAME}/${APP_NAME}:${imageTag}"
          docker_image = docker.build(imageName)
        }
      }
    }
    stage('Push Docker image') {
      steps {
        script {
          def imageTag = "${params.BRANCH_NAME}_${BUILD_NUMBER}"
          def imageName = "${DOCKERHUB_USERNAME}/${APP_NAME}:${imageTag}"
          withDockerRegistry([credentialsId: "${REGISTRY_CREDS}", url: '']) {
            docker_image.push(imageTag)
            docker_image.push('latest')
          }
        }
      }
    }
    stage('Update deployment file') {
      steps {
        script {
          def imageTag = "${params.BRANCH_NAME}_${BUILD_NUMBER}"
          sh '''
            git config user.email "mittalgaurav619@gmail.com"
            git config user.name "mittal0706"
            sed -i "s/${APP_NAME}:.*/${APP_NAME}:${imageTag}/g" deploy.yml
            git add deploy.yml
            git commit -m "Update deployment image to version ${imageTag}"
          '''
          withCredentials([gitUsernamePassword(credentialsId: 'github', gitToolName: 'Default')]) {
            sh "git push https://github.com/mittal0706/build_with_parameter.git HEAD:${params.BRANCH_NAME}"
          }
        }
      }
    }
    stage('Deploy to EKS') {
      steps {
        script {
          def imageTag = "${params.BRANCH_NAME}_${BUILD_NUMBER}"
          withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'kubenetes', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
            sh "sed -i 's/${APP_NAME}:.*/${APP_NAME}:${imageTag}/g' deploy.yml"
            sh 'kubectl apply -f deploy.yml'
          }
        }
      }
    }
  }
}
