pipeline {
  agent any
//pipeline in action
  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"//Trigger
              archive 'target/*.jar'
            }
        }   
    }
}
