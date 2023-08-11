pipeline {
  agent any
//pipeline Building
  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"//Trigger
              archive 'target/*.jar'
            }
        }   
    }
}
