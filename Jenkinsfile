pipeline {
	    agent any
	    environment {
	        branch = 'master'
	        scmUrl = 'https://github.com/ashwinmohanakrishnan/Devops-maven.git'
	        serverPort = '8080'
	        scannerHome = tool 'sonar'
	        developmentServer = 'dev-myproject.mycompany.com'
	        stagingServer = 'staging-myproject.mycompany.com'
	        productionServer = 'production-myproject.mycompany.com'
		//deploymentuser = credentials('deploymentuser')
	    }
	    
	    tools {
	    maven 'maven'
	  }
	    stages {
	        stage('checkout git') {
	            steps {
	                git branch: branch, credentialsId: 'git', url: scmUrl
	            }
	        }
			stage('build') {
	            steps {
	                echo "Building Job at ${workspace}"
	                sh 'mvn clean install'
	            }
	        }
	        stage('Sonarqube') {
	  
	             steps {
	                   withSonarQubeEnv('sonar') {
	                        sh "mvn sonar:sonar"
	                     }
	             }
	        }
	        
			stage('Sonar scan result check') {
	            steps {
	                timeout(time: 5, unit: 'MINUTES') {
	                    retry(3) {
	                        script {
	                            def qg = waitForQualityGate()
	                            if (qg.status != 'OK') {
	                                error "Pipeline aborted due to quality gate failure: ${qg.status}"
	                            }
	                        }
	                    }
	                }
	            }
	        }
	        	
	        stage ('test') {
	            steps {
	                parallel (
	                    "unit tests": { sh 'mvn test' },
	                    "integration tests": { sh 'mvn integration-test' }
	                )
	            }
	        }
	        
	     stage('Push to Artifactory') {
	         steps{
	             script {
				  // Push to Artifactory
				  def server = Artifactory.server "VM1-Artifactory"
			
				  def uploadSpec = """{
					"files": [
					  {
						"pattern": "target/*.war",
						"target": "Prasanna/${env.BUILD_NUMBER}/"
					  }
					]
				  }"""
				  // Upload to Artifactory.
				  server.upload(uploadSpec)
					 }
				 }
		   }
		stage('Pull to Artifactory') {
			 steps{
				 script {
				  // Pull to Artifactory
				  def server = Artifactory.server "Artifactory"

				  def downloadSpec = """{
					"files": [
					  {
						"pattern": "ashwin-maven-war/${env.BUILD_NUMBER}/*.war",
						"target": "artifact/"
					  }
					]
				  }"""
				  // Download from Artifactory.
				  server.download(downloadSpec)
					 }
				 }
		   }
		   stage('Deploy War to Tomcat') {
			    steps {
				echo 'Deploying....'
				sh "scp ./artifact/${env.BUILD_NUMBER}/SpringMVCHibernate.war ${deploymentuser}:/home/ubuntulogin/Docker"
			    }
			}
		}
	post {
		failure 
		{
			mail to: 'example@mindtree.com', subject: 'Pipeline failed', body: "${env.BUILD_URL}"
		}
	}
}
