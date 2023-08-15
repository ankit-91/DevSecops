pipeline {
  agent any
//Pipeline Build in the action
  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"//Trigger
              archive 'target/*.jar'
            }
        }   
      stage('Unit Tests - JUnit and Jacoco') {
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
      stage('Sonarqube SAST') {
            steps {
              sh "mvn sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.host.url=http://ec2-3-84-15-69.compute-1.amazonaws.com:9000 -Dsonar.login=2851c7840a9268478167b0c9d9b1666102c8595f"
            }
    }  } 

      stage('Docker Build and Push') {
      steps {
        withDockerRegistry([credentialsId: "docker", url: ""]) {
          sh 'printenv'
          sh 'docker build -t ankit136/numeric-app:""$GIT_COMMIT"" .'
          sh 'docker push ankit136/numeric-app:""$GIT_COMMIT""'
        }
      }
    }
  
    stage('Kubernetes Deployment - DEV') {
      steps {
        withKubeConfig([credentialsId: 'kubeconfig']) {
          sh "sed -i 's#replace#ankit136/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
          sh "kubectl apply -f k8s_deployment_service.yaml"
        }
      }
    }
    
  }

}
