pipeline{
  agent any
  environment {
	    DOCKERHUB_USERNAME = "mittal0706"
	    APP_NAME = "flask_app"
	    IMAGE_TAG = "${BUILD_NUMBER}"
	    IMAGE_NAME = "${DOCKERHUB_USERNAME}/${APP_NAME}"
	    REGISTRY_CREDS = "dockerhub"
	}
  stages{
    stage('Git Checkout') {
      steps {
        script {
          withCredentials([gitUsernamePassword(credentialsId: 'github', gitToolName: 'Default')]) {
          git branch: dev, credentialsId: 'github', url: 'https://github.com/mittal0706/build_with_parameter.git'
          }
        }
      }
    }
    stage('install dependecies'){
      steps{
        script{
          sh 'pip install -r requirements.txt'
        }
      }
    }
    stage('docker build image'){
	    steps{
	      script{
		      docker_image = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
	        }
	    }
    }

    stage('push docker image'){
	    steps{
	      script{
		      withDockerRegistry([credentialsId: "${REGISTRY_CREDS}", url: '']) {
		      docker_image.push("${BUILD_NUMBER}")
		      docker_image.push('latest')
	        }
	      }
      }
    }
	  stage('update Deployment File'){
	     when {
                expression {
                    env.GIT_BRANCH == 'origin/dev' ||
                    env.GIT_BRANCH == 'origin/uat' ||
                    env.GIT_BRANCH == 'origin/prod'
                }
            }
	  
	    steps{
		    script{  
		      sh '''
		        git config user.email "mittalgaurav619@gmail.com"
		        git config user.name "mittal0706"
		        BUILD_NUMBER=${BUILD_NUMBER}
            sed -i "s/v1/${BUILD_NUMBER}/g" deploy.yml
            git add deploy.yml
            git commit -m "Update deployment image to version ${BUILD_NUMBER}"
		        '''
            withCredentials([gitUsernamePassword(credentialsId: 'github', gitToolName: 'Default')]) {
			    sh 'git push "https://github.com/mittal0706/build_with_parameter.git" HEAD:${env.GIT_BRANCH}'
		      }
	      }
	    }
	  }
  }
}
