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

    stage('Prepare Deployment Directory') {
      steps {
        sshPublisher(publishers: [
          sshPublisherDesc(
            configName: 'dockerhost', // Configuration SSH correcte pour le serveur Docker
            transfers: [
              sshTransfer(execCommand: "/bin/rm -rf /opt/docker/*.war")
            ]
          )
        ])
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
                remoteDirectory: '//opt/docker'
              )
            ]
          )
        ])
      }
    }

    stage('Deploy and Log') {
      steps {
        sshPublisher(publishers: [
          sshPublisherDesc(
            configName: 'dockerhost',
            transfers: [
              sshTransfer(
                execCommand: "cd /opt/docker;docker build -t regapp:v1 .;docker stop registerapp;docker rm registerapp;docker run -d --name registerapp -p 8081:8080 regapp:v1;"
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
