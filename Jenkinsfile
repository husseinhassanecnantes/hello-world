pipeline {
  agent any
  tools {
    maven 'maven.3.9.7'
  }
  stages {
    stage('Compile') {
      steps {
        sh 'mvn compile'
        sh 'echo "Compile successful"'
      }
    }

    stage('Test') {
      steps {
        sh 'mvn test'
        sh 'echo "Test successful"'
      }
    }

    stage('Build') {
      steps {
        sh 'mvn clean'
        sh 'mvn install'
        sh 'echo "Build successful"'
      }
    }

    stage('Transfer Artifact') {
      steps {
        sshPublisher(publishers: [
          sshPublisherDesc(
            configName: 'dockerhost',
            transfers: [
              sshTransfer(
                sourceFiles: 'webapp/target/*.war',
                removePrefix: 'webapp/target',
                remoteDirectory: '/opt/docker'
              )
            ]
          )
        ])
      }
    }

    stage('Build Docker Image') {
      steps {
        sshPublisher(publishers: [
          sshPublisherDesc(
            configName: 'dockerhost',
            transfers: [
              sshTransfer(
                sourceFiles: '',
                execCommand: '''
                  cd /opt/docker;
                  docker build -t regapp:v1 .;
                '''
              )
            ]
          )
        ])
      }
    }

    stage('Stop and Remove Existing Container') {
      steps {
        sshPublisher(publishers: [
          sshPublisherDesc(
            configName: 'dockerhost',
            transfers: [
              sshTransfer(
                sourceFiles: '',
                execCommand: '''
                  docker stop registerapp || true;
                  docker rm registerapp || true;
                '''
              )
            ]
          )
        ])
      }
    }

    stage('Run Docker Container') {
      steps {
        sshPublisher(publishers: [
          sshPublisherDesc(
            configName: 'dockerhost',
            transfers: [
              sshTransfer(
                sourceFiles: '',
                execCommand: '''
                  docker run -d --name registerapp -p 8081:8080 regapp:v1;
                '''
              )
            ]
          )
        ])
      }
    }
  }
  post {
    failure {
      sh 'echo "Failure Failure"'
    }
  }
}
