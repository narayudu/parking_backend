pipeline {
   agent any
	stages {
      stage('Git Checkout') {
         steps {
            git 'https://github.com/narayudu/parking_backend.git'
		}
	}
	 
      		
	stage('Build') {
		steps {
			withSonarQubeEnv('sonar') {
				sh '/opt/maven/bin/mvn clean verify sonar:sonar -Dmaven.test.skip=true'
			}
		}
	}
		
       stage('Test & Jacoco Static Analysis') {
	    junit '**/target/surefire-reports/TEST-*.xml'   
            
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
      }}
   
   			
	stage ('Deploy') {
		steps {
			sh '/opt/maven/bin/mvn clean deploy -Dmaven.test.skip=true'
		}
	}
	stage ('Release') {
		steps {
			sh 'export JENKINS_NODE_COOKIE=dontkillme ;nohup java -jar $WORKSPACE/target/*.jar &'
		}
	}
		
      	
 /*     stage('TF Plan') {
             steps {
             sh 'terraform init'
           sh 'terraform plan -out myplan'
    
       }
     }
		
   stage('TF Apply') {
      steps {
          sh 'terraform apply -input=false myplan'
        }
      }	 */	
}
	post {
        always {
            emailext body: "${currentBuild.currentResult}: Project Name : ${env.JOB_NAME} Build ID : ${env.BUILD_NUMBER}\n\n Approval Link :  ${env.BUILD_URL}", recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']], subject: 'Test'
        }
    }

}
