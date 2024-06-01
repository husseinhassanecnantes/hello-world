pipeline {
  agent any
  tools {
    maven 'maven.3.9.7' 
  }
  stages {
	stage ('Compile') {
	  steps {
		sh 'mvn compile'
		sh 'echo "test successful"'
		}	
	}

	stage ('Test') {  
	  steps{
		sh 'mvn test'
		sh 'echo "Test successful"'
		} 
	}

	stage ('Build') {
	  steps {
		sh 'mvn clean'
		sh 'mvn install'
		sh 'echo "Build successful"'
	  }
	}
		
	stage ('Deploy') {
	  steps {
		script {
		  deploy adapters: [tomcat9(credentialsId: 'tomcat_deployer', path: '', url: 'http://15.237.110.188:8080/')], contextPath: 'webapp', onFailure: false, war: '**/*.war' 
		}
	  }
	}	
  }
  post {
    failure {  
		sh 'echo "Failure Failure"'
	}
  }

}