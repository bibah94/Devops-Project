pipeline {
	
  agent any
  
  environment {
    PATH = "/opt/maven/bin:$PATH"
    scannerHome = tool 'sonar-scanner'
  }
  
  stages {

    stage('Cloning Project') {
      steps {
        git(url: 'https://github.com/bibah94/Devops-Project.git', branch: 'master')
      }
    }

    stage('Build app'){
      steps {
        sh "mvn -Dmaven.test.failure.ignore=true package"
      }
    }
	  
    stage('Static Code Analysis') {
      agent any
      steps {
        withSonarQubeEnv(envOnly: true, installationName: 'sonarqube-server', credentialsId: '4f92fd01-ca54-4b3d-b1fd-c96a30aa2e2a') {
          sh "mvn clean package sonar:sonar"
        }       
      }
    }

    stage('Build Docker Image, Push to Nexus and Deploy to Kubernetes'){
      steps {
        sshPublisher(publishers: [sshPublisherDesc(
          configName: 'ansible-server', 
          transfers: [sshTransfer(
            execCommand: 'ansible-playbook -i /opt/docker/hosts /opt/docker/devops-docker-image.yml', 
            remoteDirectory: '//opt//docker', 
            removePrefix: 'target', 
            sourceFiles: 'target/*.war')], 
            verbose: false
        )])
      }
    }
  }
post{
    success{
	slackSend  channel: '#jenkins',
		   color: 'good',
		   message: "Build ${env.BUILD_NUMBER}, success: ${currentBuild.fullDisplayName}."
    }
    failure{
	slackSend  channel: '#jenkins',
		   color: 'danger',
		   message: "Build ${env.BUILD_NUMBER}, failed: ${currentBuild.fullDisplayName}."	
    }
    always {
      emailext body: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}",
           recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']],
           to: 'habibndiaye08@gmail.com',
           subject: "Jenkins Build ${currentBuild.currentResult}: Job ${env.JOB_NAME}"
    }
  }
}
