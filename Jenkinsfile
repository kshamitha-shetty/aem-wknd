def mavenPom
pipeline {
    agent any
    tools {
        jdk 'jdk-8'
        maven 'maven-3.6.3'
    }
    
    stages {

        stage('Git Checkout') {
            steps {
                echo 'Checking out git repository'
		        git 'https://github.com/archna1402/aem-wknd.git'
            }
        }
		

        stage('Build and Test') {
            steps {
                //input ('Do you want to proceed?')
                script {
                    try {
                        sh 'mvn clean install' 
                        echo "Build completed. RESULT: ${currentBuild.currentResult}"
                    } catch (Throwable e) {
                        echo "The current build has failed. Please check logs."
                        error "ERROR! Stop pipeline excution!"
                    }
                }
            }
        }
		
		stage('SonarQube Analysis') {
		environment {
        scannerHome = tool 'SonarQubeScanner'
    }
            steps {
                echo "SonarQube Analysis"
				withSonarQubeEnv(credentialsId: 'loyltydemo', installationName: 'sonarqualitygate'){
           sh "${scannerHome}/bin/sonar-scanner"
		      //sh 'mvn clean package sonar:sonar'
        }
        timeout(time: 10, unit: 'MINUTES') {
            waitForQualityGate abortPipeline: true
			
        }
            }
        }

		stage('Upload Artifacts to Nexus') {
		    
				steps{
					script{
						mavenPom = readMavenPom file: 'pom.xml'
						nexusArtifactUploader artifacts: 
							[[artifactId: 'aem-guides-wknd',
							    classifier: 'all', 
								file: "/var/lib/jenkins/.m2/repository/com/adobe/aem/guides/aem-guides-wknd.all/${mavenPom.version}/aem-guides-wknd.all-${mavenPom.version}.zip",  							
								type: 'zip']],
								credentialsId: 'nexus3', 
								groupId: 'com.adobe.aem.guides', 
								nexusUrl: '3.6.165.101:8081', 
								nexusVersion: 'nexus3', 
								protocol: 'http', 
								repository: 'aem-wknd-snapshots', 
								version: "${mavenPom.version}"
						}
					}
				}	

      stage('Deploy to AEM server') {
			steps { 
					echo "Deploy to AEM Author Instance"
					script{
						mavenPom = readMavenPom file: 'pom.xml'
						sh "curl -u admin:admin -F file=@'/var/lib/jenkins/.m2/repository/com/adobe/aem/guides/aem-guides-wknd.all/${mavenPom.version}/aem-guides-wknd.all-${mavenPom.version}.zip' -F name='aem-guides-wknd.all-${mavenPom.version}' -F force=true -F install=true http://localhost:4502/crx/packmgr/service.jsp"
						echo "Deploy to AEM Publish Instance"
						sh "curl -u admin:admin -F file=@'/var/lib/jenkins/.m2/repository/com/adobe/aem/guides/aem-guides-wknd.all/${mavenPom.version}/aem-guides-wknd.all-${mavenPom.version}.zip' -F name='aem-guides-wknd.all-${mavenPom.version}' -F force=true -F install=true http://localhost:4503/crx/packmgr/service.jsp"
					}
			}
     }
	 
	}

    post {
        success {
            sh '/var/lib/jenkins/scripts/clear_dispatcher_cache.sh'
            echo "Build Success : : ${env.BUILD_NUMBER}"
			echo "RESULT: ${currentBuild.currentResult}"
        }
        failure {
            echo "Build Failure : ${env.BUILD_NUMBER}"
			echo "RESULT: ${currentBuild.currentResult}"
        }
		always {
			emailext attachLog: true,
            subject: "Status of Build : ${env.JOB_NAME} #${env.BUILD_NUMBER} - ${currentBuild.result}",
            body: "${env.JOB_NAME} #${env.BUILD_NUMBER} has result ${currentBuild.result}.\n\nView the console at: ${env.BUILD_URL}. Build log is attached.\n\n",
			to: 'archna@epsilon.com harikrishnan.suresh@epsilonconversant.com eshan.verma@epsilonconversant.com srinivas.garimella@epsilonconversant.com kshamitha@epsilonconversant.com'
        }
    }
}