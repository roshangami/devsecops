pipeline {
	agent any

	stages {
		stage ('Initialize') {
		      steps {
		        sh '''
                		echo "PATH = ${PATH}"
		                echo "M2_HOME = ${M2_HOME}"
		            ''' 
      			}
    		}
		stage ('Check Secrets') {
     			steps {
			      sh 'trufflehog3 https://github.com/roshangami/devsecops.git -f json -o truffelhog_output.json || true'
			}
    		}
		
	stage ('Software Composition Analysis') {
            steps {
                dependencyCheck additionalArguments: ''' 
                    -o "./" 
                    -s "./"
                    -f "ALL" 
                    --prettyPrint''', odcInstallation: 'owasp-dc'

                dependencyCheckPublisher pattern: 'dependency-check-report.xml'
            }
        }
		
	
	stage ('SAST - SonarQube') {
	      steps {
        	withSonarQubeEnv('sonarqube') {
	          sh 'mvn sonar:sonar'
				}
	      	}
    	}
	
	
	stage ('SAST - SemGrep') {
	      steps {
        	sh 'sudo docker run --rm -v "${PWD}:/src" returntocorp/semgrep semgrep --config=auto --output semgrep_output.json --json'
		//sh './semgrep_report.sh'
	      	}
    	}
		
	stage('Compile and Build'){
		steps{
			sh 'mvn clean install -DskipTests'
			}
		}
	stage ('Deploy to server') {
            steps {
        	   sshagent(['application-server']) {
                	sh 'scp -o StrictHostKeyChecking=no /var/lib/jenkins/workspace/webgoat-devsecops/webgoat-server/target/webgoat-server-v8.2.0-SNAPSHOT.jar ubuntu@172.31.27.167:/WebGoat'
			sh 'ssh -o  StrictHostKeyChecking=no ubuntu@172.31.27.167 "nohup java -jar /WebGoat/webgoat-server-v8.2.0-SNAPSHOT.jar --server.address=0.0.0.0 --server.port=8080 &"'
           		}      
           	}     
    	}
		
    }
}
