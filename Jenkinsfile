pipeline { 

  agent any 

    environment { 

      deploymentName = "devsecops" 

      containerName = "devsecops-container" 

      serviceName = "devsecops-svc" 

      imageName = "ankit136/numeric-app:${GIT_COMMIT}" 

      applicationURL = "http://ec2-18-207-188-19.compute-1.amazonaws.com" 

      applicationURI = "/increment/99" 

 /*     COSIGN_PASSWORD=credentials('cosign-password') 

      COSIGN_PRIVATE_KEY=credentials('cosign-private-key') 
*/
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

      

  /*   stage('SonarQube - SAST') { 

      steps { 

        withSonarQubeEnv('SonarQube') { 

          sh "mvn sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.host.url=http://ec2-54-226-201-238.compute-1.amazonaws.com:9000 -Dsonar.login=684b03b2b5e145e1dddfcf6ac3b4e6b27eac1eb2" 

        } 

        timeout(time: 2, unit: 'MINUTES') { 

          script { 

            waitForQualityGate abortPipeline: true 

          } 

        } 

      } 

    } 
*/
             

    

  

  /*  stage('Vulnerability Scan of base Image of Application Image') { 

      steps { 

          sh "bash trivy-docker-image-scan.sh" 

      } 

      } 

    */

      stage('OPA-Docker') {

	steps {
	  sh 'docker run --rm -v $(pwd):/project openpolicyagent/conftest test --policy opa-docker-security.rego Dockerfile'
		
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

    

      } 

    }   

    }  

   
     

 /*   stage('App Image Signing in DockerHub') { 

      steps { 

         

           withDockerRegistry([credentialsId: "docker-hub", url: ""]) { 

         sh 'cosign sign --key $COSIGN_PRIVATE_KEY "ankit136/numeric-app:${GIT_COMMIT}" -y' 

           } 

      } 

    } 

  */     
    stage('Vulnerability Scan - Kubernetes') {
      steps {
        parallel(
          "OPA Scan": {
            sh 'docker run --rm -v $(pwd):/project openpolicyagent/conftest test --policy opa-k8s-security.rego k8s_deployment_service.yaml'
          },
          "Kubesec Scan": {
            sh "bash kubesec-scan.sh"
          }
        )
      }
    }

	  
    /*  stage('Kubernetes Deployment') { 

      steps { 

        withKubeConfig([credentialsId: 'kubeconfig']) { 

          sh "sed -i 's#replace#ankit136/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml" 

          sh "kubectl apply -f k8s_deployment_service.yaml" 

          echo "The value of the variable is ${GIT_COMMIT}" 

        } 

      } 

    } 
*/
   stage('K8S Deployment - DEV') {
      steps {
        parallel(
          "Deployment": {
            withKubeConfig([credentialsId: 'kubeconfig']) {
              sh "bash k8s-deployment.sh"
            }
          },
          "Rollout Status": {
            withKubeConfig([credentialsId: 'kubeconfig']) {
              sh "bash k8s-deployment-rollout-status.sh"
            }
          }
        )
      }
    }

stage('Integration Tests - DEV') {
      steps {
        script {
          try {
            withKubeConfig([credentialsId: 'kubeconfig']) {
              sh "bash integration-test.sh"
            }
          } catch (e) {
            withKubeConfig([credentialsId: 'kubeconfig']) {
              sh "kubectl -n default rollout undo deploy ${deploymentName}"
            }
            throw e
          }
        }
      }
    }  


   /* stage('OWASP ZAP - DAST') { 

      steps { 

        withKubeConfig([credentialsId: 'kubeconfig']) { 

          sh 'bash zap.sh' 

        } 

	

      } 

    } 
   */
    
    stage('K8S Deployment - PROD') {
      steps {
        parallel(
          "Deployment": {
            withKubeConfig([credentialsId: 'kubeconfig']) {
              sh "sed -i 's#replace#${imageName}#g' k8s_PROD-deployment_service.yaml"
              sh "kubectl -n prod apply -f k8s_PROD-deployment_service.yaml"
            }
          },
          "Rollout Status": {
            withKubeConfig([credentialsId: 'kubeconfig']) {
              sh "bash k8s-PROD-deployment-rollout-status.sh"
            }
          }
        )
      }
    }
	
   
  } 

/*   post { 

    always {  

    publishHTML([allowMissing: false, alwaysLinkToLastBuild: true, keepAll: true, reportDir: 'owasp-zap-report', reportFiles: 'zap_report.html', reportName: 'OWASP ZAP HTML Report', reportTitles: 'OWASP ZAP HTML Report', useWrapperFileDirectly: true]) 

    } 

     

   }  

  
*/
} 
