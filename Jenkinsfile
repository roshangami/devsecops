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
		sh './semgrep_report.sh'
	      	}
    	}
		
	stage('Compile and Build'){
		steps{
			sh 'mvn clean install -DskipTests'
			}
		}
		
	}
}
