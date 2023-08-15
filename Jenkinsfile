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
              withSonarQubeEnv('SonarQube') {
              sh "mvn sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.host.url=http://ec2-3-86-24-84.compute-1.amazonaws.com:9000 -Dsonar.login=0b30ed6d8d32300d09d54dbdc977d319ebf1517e"
            }
            timeout(time: 2, unit: 'MINUTES') {
              script {
                waitForQualityGate abortPipeline: true
              }  
            }
            }
      }

      //stage('Vulnerability Scan - Docker ') {
      //steps {
        //sh "mvn dependency-check:check"
      //}
      //post {
        //always {
          //dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
        //}
      //}
    //}

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
