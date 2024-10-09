pipeline {
	agent any

    environment{
        registry = "waughananda/vprofileapp"
        registryCredential = "dockerhub"
    }
	tools {
			maven "MAVEN3"
			jdk "OracleJDK8"
		}
	stages {
		stage('build'){
			steps{
				sh 'mvn clean install -DskipTests'
			}
			post {
				success{
					echo 'Archiving artifacts now.'
					archiveArtifacts artifacts: '**/*.war'
				}
			}
		}
		stage('UNIT TESTS'){
			steps {
				sh 'mvn test'
			}
		}
		
		stage ('Checkstyle Analysis'){
			steps{
				sh 'mvn checkstyle:checkstyle'
			}
			post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
		}
		
		stage('Build App Image'){
			steps{
				script{
					dockerImage=docker.build registry + ":V$BUILD_NUMBER"
				}
			}
		}
		
		stage('Upload App Image'){
			steps{
				script{
					docker.withRegistry( '', registryCredential){
						dockerImage.push("V$BUILD_NUMBER")
						dockerImage.push('latest')
					}
				}
			}
		}
		
        stage('Remove unused docker images'){
            steps{
                sh "docker rmi $registry:V$BUILD_NUMBER"
            }
        }
		
		stage ('Sonar Analysis'){
			environment {
				scannerHome = tool 'sonar4.7'
			}
			steps{
				withSonarQubeEnv('sonar'){
					sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                            -Dsonar.projectName=vprofile \
                            -Dsonar.projectVersion=1.0 \
                            -Dsonar.sources=src/ \
							-Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                            -Dsonar.junit.reportsPath=target/surefire-reports/ \
							-Dsonar.jacoco.reportsPath=target/jacoco.exec \
							-Dsonar.java.checkstyle.reportsPath=target/checkstyle-result.xml '''
				}
			}
		}
		stage('Quality Gate') {
            steps {
                // Wait for SonarQube analysis to be completed and check quality gate
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        
        
        stage('Kubernetes Deploy'){
            agent{label 'SILVER'}
                steps{
                    sh "helm upgrade --install --force vproifle-stack helm/vprofilecharts --set appimage=${registry}:${BUILD_NUMBER} --namespace prod"                
				/*	sh "date >> testdate.txt" */
				}

        }	
		
	}
}