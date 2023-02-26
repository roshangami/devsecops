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
		stage ('Check secrets') {
     			steps {
			      sh 'trufflehog3 https://github.com/roshangami/devsecops.git -f json -o truffelhog_output.json || true'
				      }
    			}
		
	stage ('Software composition analysis') {
            steps {
                dependencyCheck additionalArguments: ''' 
                    -o "./" 
                    -s "./"
                    -f "ALL" 
                    --prettyPrint''', odcInstallation: 'owasp-dc'

                dependencyCheckPublisher pattern: 'dependency-check-report.xml'
            }
        }
		
	stage('Compile Stage'){
		steps{
			sh 'mvn clean install -DskipTests'
			}
		}
		
	stage ('Static analysis') {
	      steps {
        	withSonarQubeEnv('sonarqube') {
	          sh 'mvn sonar:sonar'
				}
	      	}
    	}
	}
}
