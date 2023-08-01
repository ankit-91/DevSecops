pipeline {
  agent any
//pipeline in actions Yooo
  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"//Trigger
              archive 'target/*.jar'
            }
        }   
    }
}
