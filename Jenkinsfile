pipeline {
    agent any
    environment {
        DOCKER_IMAGE_NAME = "ahmedatya11/java-test-app"
    }

    stages {
      //Cloning CI Repo to git the last version of the code
      stage('Clone repository') {  
        steps {
        checkout scm
    }
      }
      // Build Dockerfile and Rename it with the Dockerhub-reg Account name and Add the commmit number as tag name 
      stage('Test and Build Docker Image') {
         steps {       
         script {
                    env.GIT_COMMIT_REV = sh (script: 'git log -n 1 --pretty=format:"%h"', returnStdout: true)
                    customImage = docker.build("${DOCKER_IMAGE_NAME}:${GIT_COMMIT_REV}")
                    }
                    }                           
     }
     // push Docker Image to the registry with the modified name 
    stage('Deploy Image') {
      steps{
        script {
          docker.withRegistry( '', "docker-cred" ) {
            sh 'docker push ${DOCKER_IMAGE_NAME}:${GIT_COMMIT_REV}'
          }
        }
      }
    }
    // Trigger  the CD job at Jenkins
        stage('Trigger CD job ') {
                steps {
                echo "triggering CD"
                build job: 'CD', parameters: [string(name: 'GIT_COMMIT_REV', value: env.GIT_COMMIT_REV)]
        }
        }      
  }
 
}
