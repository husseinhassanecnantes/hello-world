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
              sshTransfer(
                sourceFiles: '', // Pas de fichier à transférer ici
                execCommand: '''
                  # Suppression du fichier webapp.war existant dans /opt/docker
                  rm -f /opt/docker/webapp.war
                '''
              )
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
                sourceFiles: 'webapp/target/webapp.war', // Chemin correct du fichier webapp.war
                remoteDirectory: '/opt/docker' // Assurez-vous que le fichier est transféré dans /opt/docker
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
                sourceFiles: '', // Pas de fichier à transférer ici
                execCommand: '''
                  # Vérification du transfert
                  ls -la /opt/docker;
                  echo "Verify that webapp.war was transferred to /opt/docker";

                  # Construction de l'image Docker
                  cd /opt/docker;
                  docker build -t regapp:v1 .;
                  echo "Docker image build complete";
                  docker images; # Ajouté pour vérifier la création de l'image

                  # Arrêt et suppression du conteneur existant
                  docker stop registerapp || true;
                  docker rm registerapp || true;
                  echo "Existing container stopped and removed";
                  docker ps -a; # Ajouté pour vérifier les conteneurs

                  # Exécution du nouveau conteneur Docker
                  docker run -d --name registerapp -p 8081:8080 regapp:v1;
                  docker ps -a; # Ajouté pour vérifier que le conteneur a démarré
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
