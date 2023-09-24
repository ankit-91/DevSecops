pipeline {
  agent any
    environment {
    //deploymentName = "devsecops"
    //containerName = "devsecops-container"
    //serviceName = "devsecops-svc"
    //imageName = "ankit136/numeric-app:${GIT_COMMIT}"
    //applicationURL = "http://ec2-3-236-105-192.compute-1.amazonaws.com"
    //applicationURI = "/increment/99"
    COSIGN_PASSWORD=credentials('cosign-password')
    COSIGN_PRIVATE_KEY=credentials('cosign-private-key')
  } 
//Pipeline Building in the action
  stages {
      stage('Build Artifact-Maven') {
            steps {
              sh "mvn clean package -DskipTests=true"//Trigger
              archive 'target/*.jar'
            }
        }   
      stage('Unit tests with Jacoco') {      
   
      steps {
        sh "mvn test"
      }
      post {
        always {
          junit 'target/surefire-reports/*.xml'
          jacoco execPattern: 'target/jacoco.exec'
        }
      }
    }
     
  /*  stage('SonarQube - SAST') {
      steps {
        withSonarQubeEnv('SonarQube') {
          sh "mvn sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.host.url=http://ec2-54-147-158-76.compute-1.amazonaws.com:9000 -Dsonar.login=b61699de4a3f1c5d8d51da2d6636e0ec6631de73"
        }
        timeout(time: 2, unit: 'MINUTES') {
          script {
            waitForQualityGate abortPipeline: true
          }
        }
      }
    }
    */        
   

    stage('Vulnerability Scan of base Image of Application Image') {
      steps {
          sh "bash trivy-docker-image-scan.sh"
      }
      }
   
      stage('Docker Image Build') {
      steps {
         sh 'sudo docker build -t ankit136/numeric-app:""$GIT_COMMIT"" .'
        }
    }

    stage('Docker App Image push to Dockerhub') {
      steps {
          withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
           sh 'docker push ankit136/numeric-app:""$GIT_COMMIT""'
         //sh 'cosign sign --key $COSIGN_PRIVATE_KEY ankit136/numeric-app:""$GIT_COMMIT"" -y'
      }
    }  
    } 

    stage('App Image Signing in DockerHub') {
      steps {
        //sh 'cosign sign --key $COSIGN_PRIVATE_KEY ankit136/numeric-app:""$GIT_COMMIT"" -y'
          sh 'cosign sign --key $COSIGN_PRIVATE_KEY **** ankit136/numeric-app:""$GIT_COMMIT"" -r docker.io -y'
      }
    }
      
 /*   stage('Kubernetes Deployment - DEV') {
      steps {
        withKubeConfig([credentialsId: 'kubeconfig']) {
          sh "sed -i 's#replace#ankit136/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
          sh "kubectl apply -f k8s_deployment_service.yaml"
        }
      }
    }

    stage('OWASP ZAP - DAST') {
      steps {
        withKubeConfig([credentialsId: 'kubeconfig']) {
          sh 'bash zap.sh'
        }
      }
    }
    
  }
   post {
    always { 
    publishHTML([allowMissing: false, alwaysLinkToLastBuild: true, keepAll: true, reportDir: 'owasp-zap-report', reportFiles: 'zap_report.html', reportName: 'OWASP ZAP HTML Report', reportTitles: 'OWASP ZAP HTML Report', useWrapperFileDirectly: true])
    }
 */   
   } 

}
