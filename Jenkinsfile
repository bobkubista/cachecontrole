pipeline {
  agent any
  tools {
    maven 'maven'
  }
  stages {
    stage('compile') {
      steps {
        sh 'mvn clean compile'
        stash 'compile'
      }
    }
    stage('validate') {
      steps {
        unstash 'compile'
        sh 'mvn -B -T 1C validate -am'
      }
    }
    stage('test') {
      parallel {
        stage('test') {
          steps {
            unstash 'compile'
            sh 'mvn test'
            stash 'test'
          }
        }
        stage('integration test') {
          steps {
            unstash 'compile'
            sh 'mvn integration-test'
            stash 'it-test'
          }
        }
      }
    }
    stage('deploy') {
      steps {
        unstash 'test'
        unstash 'it-test'
        timeout(time: 10, unit: 'MINUTES') {
          input(message: 'Deploy', id: 'deply')
          sh 'mvn -T 1C -f services/rest-services/spring-services/user/user-service/pom.xml cargo:undeploy cargo:deploy -X '
          sh 'mvn -T 1C -f services/rest-services/cdi-services/datagathering/datagathering-rest-service/pom.xml cargo:undeploy cargo:deploy -X '
        }

        stash 'deploy'
      }
    }
    stage('performance test') {
      steps {
        sh 'mvn verify -P performance-test -T 1C -am'
        junit '**/*.jtl'
      }
    }
    stage('sonar') {
      steps {
        unstash 'deploy'
        sh 'mvn sonar:sonar -P sonar -am -T 1C '
      }
    }
    stage('nexus') {
      steps {
        sh 'mvn deploy -T 1C -am'
      }
    }
    stage('release') {
      steps {
        sh 'mvn -T 1C -am -e -X release:prepare'
        sh 'mvn -T 1C -am -DdryRun=true -e -X release:perform'
      }
    }
  }
  post {
    always {
      echo 'Build stage complete'

    }

    failure {
      echo 'Build failed'
      mail(body: 'build failed', subject: 'Build failed!', to: 'bob@bobkubista.com')

    }

    success {
      echo 'Build succeeded'
      mail(body: 'build succeeded', subject: 'Build Succeeded', to: 'bob@bobkubista.com')

    }

  }
}
