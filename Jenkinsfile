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
		        git 'https://github.com/kshamitha-shetty/sonar-validation.git'
            }
        }
		stage('Build and Test') {
            steps {
                //input ('Do you want to proceed?')
                script {
                    try {
                        sh 'mvn clean package' 
                        echo "Build completed. RESULT: ${currentBuild.currentResult}"
                    } catch (Throwable e) {
                        echo "The current build has failed. Please check logs."
                        error "ERROR! Stop pipeline excution!"
                    }
                }
            }
        }
		stage('Sonarqube') {
    environment {
        scannerHome = tool 'SonarQubeScanner'
    }
    steps {
        withSonarQubeEnv(credentialsId: 'loyltydemo', installationName: 'sonarqualitygate'){
            sh "${scannerHome}/bin/sonar-scanner"
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