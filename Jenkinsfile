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
            configName: 'dockerhost',
            transfers: [
              sshTransfer(
                sourceFiles: '',
                execCommand: '''
                  # Ensure the deployment directory is clean
                  rm -f //opt//docker//webapp.war
                '''
              )
            ]
          )
        ])
      }
    }

    stage('Transfer and Deploy') {
      steps {
        sshPublisher(publishers: [
          sshPublisherDesc(
            configName: 'dockerhost',
            transfers: [
              sshTransfer(
                sourceFiles: 'webapp/target/*.war',
                removePrefix: 'webapp/target',
                remoteDirectory: '/opt/docker',
                execCommand: '''
                  # Verify the transfer
                  ls -la /opt/docker;
                  echo "Verify that webapp.war was transferred to /opt/docker";

                  # Build Docker image
                  cd /opt/docker;
                  docker build -t regapp:v1 .;
                  echo "Docker image build complete";

                  # Stop and remove existing container
                  docker stop registerapp || true;
                  docker rm registerapp || true;
                  echo "Existing container stopped and removed";

                  # Run Docker container
                  docker run -d --name registerapp -p 8081:8080 regapp:v1;
                  docker ps -a;
                  echo "New container started";
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
