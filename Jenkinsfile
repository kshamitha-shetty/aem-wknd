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
		        git 'https://github.com/kshamitha-shetty/aem-wknd.git'
            }
        }
		stage('Sonarqube') {
    environment {
        scannerHome = tool 'SonarQubeScanner'
    }
    steps {
        withSonarQubeEnv(credentialsId: 'loyltydemo', installationName: 'sonarqualitygate'){
           // sh "${scannerHome}/bin/sonar-scanner"
		      sh 'mvn clean package sonar:sonar'
        }
        timeout(time: 10, unit: 'MINUTES') {
            waitForQualityGate abortPipeline: true
			mail to: 'kshamitha@epsilonconversant.com',
            subject: "Status of Sonar Analysis",
            body: "Sonar Analysis Is Success"
			
        }
    }
}

}

}