pipeline {
    agent any
    environment {
        DOCKER_IMAGE_NAME = "ahmedatya11/java-test-app"
    }

    stages {
      stage('Clone repository') {  
        steps {
        checkout scm
    }
      }


      stage('Test and Build Docker Image') {
         steps {       
         script {
                    env.GIT_COMMIT_REV = sh (script: 'git log -n 1 --pretty=format:"%h"', returnStdout: true)
                    customImage = docker.build("${DOCKER_IMAGE_NAME}:${GIT_COMMIT_REV}")
                    }
                    }                           
     }
    //  stage('Login') {
    //   steps {
    //     sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
    //   }
    // }
    // stage('Push') {
    //   steps {
    //     sh 'docker push ahmedatya11/$customImage'
    //   }
    // }
    stage('Deploy Image') {
      steps{
        script {
          docker.withRegistry( '', "docker-cred" ) {
            sh 'docker push ${DOCKER_IMAGE_NAME}:${GIT_COMMIT_REV}'
          }
        }
      }
    }
    //   stage('Push image'){
    //     steps {
    //     withDockerRegistry([ credentialsId: "docker-cred", url: "" ]) {
    //     sh 'docker push <image>'
    //     }
    // }                     
    // }             
//      stage('Push Docker Image') {
//       steps {  
            
//                 script {
//                     docker.withRegistry('https://registry.hub.docker.com', 'docker') {
//                         customImage.push("${GIT_COMMIT_REV}")
//                     //    customImage.push("latest")
                    
//                     }
//                 }
//             }
//    }
   


        stage('Trigger CD job ') {
                steps {
                echo "triggering CD"
                build job: 'CD', parameters: [string(name: 'GIT_COMMIT_REV', value: env.GIT_COMMIT_REV)]
        }
        }      
  }
 
}
