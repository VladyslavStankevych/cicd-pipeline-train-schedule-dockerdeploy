pipeline {
  agent any
  stages {
    stage ('Build') {
      steps {
        echo "Building automation"
        sh './gradlew build --no-daemon'
        archiveArtifacts artifacts: 'dist/trainSchedule.zip'
      }
    }
    stage ('Building Docker Image') {
     when {
       branch 'master'
     }
     steps {
        script {
          app = docker.build("vladyslavs/train-schedule")
          app.inside {
            sh 'echo $(curl localhost:8080)'
          }
        }
      }
     } 
     stage ('Pushing Image to Docker Hub') {
       when {
         branch 'master'
       }
       steps {
         script {
           docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
             app.push("${env.BUILD_NUMBER}")
             app.push("latest")
           }
	 }
       }
     }
     stage ('Deploying to Production') {
       when {
         branch 'master'
       }
       steps {
         input 'Deploy to Production?'
         milestone(1)
         withCredentials([usernamePassword(credentialsId: 'deploy', usernameVariable: 'USERNAME', passwordVaribale: 'USERPASS')]) {
           script {
             sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@prod_ip \"docker pull vladyslavs/train-schedule:${env.BUILD_NUMBER}] \""
             try {
               sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@prod_ip \"docker stop train-schedule \""
               sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@prod_ip \"docker rm train-schedule \""
             } catch (err) {
                echo "caught error: $err"
             }
             sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@prod_ip \"docker container run --restart always --name train-schedule -p 8080:8080 -d vladyslavs/train-schedule:${BUILD_NUMBER}\""
           }
         }
       }
     }
   }
 }     
   
