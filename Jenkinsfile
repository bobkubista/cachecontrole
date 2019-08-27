pipeline {
  agent any
  stages {
    stage('compile') {
      steps {
        sh 'mvn clean compile'
      }
    }
    stage('test') {
      parallel {
        stage('test') {
          steps {
            sh 'mvn test'
          }
        }
        stage('integration test') {
          steps {
            sh 'mvn integration-test'
          }
        }
      }
    }
    stage('deploy') {
      steps {
        sh 'mvn -T 1C -f services/rest-services/spring-services/user/user-service/pom.xml cargo:undeploy cargo:deploy -X '
        sh 'mvn -T 1C -f services/rest-services/cdi-services/datagathering/datagathering-rest-service/pom.xml cargo:undeploy cargo:deploy -X '
      }
    }
    stage('sonar') {
      steps {
        sh 'mvn sonar:sonar -P sonar -am -T 1C '
      }
    }
    stage('nexus') {
      steps {
        sh 'mvn deploy -T 1C -am'
      }
    }
    stage('mail') {
      steps {
        mail(subject: 'build', body: 'build is done')
      }
    }
  }
}