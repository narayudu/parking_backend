pipeline {
   agent any
	stages {
      stage('Git Checkout') {
         steps {
            git 'https://github.com/narayudu/parking_backend.git'
		}
	}
	
     stage('Build Approval Message'){
           steps{
           slackSend baseUrl: 'https://hooks.slack.com/services/', 
           channel: 'cicd', 
           color: 'good', 
           message: 'Waiting for Build Approval', 
           tokenCredentialId: 'slack_notify', 
           username: 'narayudu kathula'
        }
   }
      
	stage('Build Approval'){
           input {
              message "Should we continue?"
               }
              steps {
              echo "Continuing with deployment"
      }
    }
	    		
	stage('Build') {
		steps {
			withSonarQubeEnv('sonar') {
				sh '/opt/maven/bin/mvn clean verify sonar:sonar -Dmaven.test.skip=true  ' 
			}
		}
	}
						
	stage("Quality Gate") {
            steps {
              timeout(time: 2, unit: 'MINUTES') {
                waitForQualityGate abortPipeline: true
              }
            }
          }
	
			
	stage ('OWASP Dependency-Check Vulnerabilities') {
            steps {
            sh '/opt/maven/bin/mvn dependency-check:check'
	    dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
            }
	}
    			
	stage ('Deploy') {
		steps {
			sh '/opt/maven/bin/mvn clean deploy -Dmaven.test.skip=true'
		}
	}
	}	
	post {
        always {
	 	
            emailext body: "${currentBuild.currentResult}: Project Name : ${env.JOB_NAME} Build ID : ${env.BUILD_NUMBER}\n\n Approval Link :  ${env.BUILD_URL}", recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']], subject: 'Test'
        }
    }

}
