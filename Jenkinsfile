pipeline {
    agent {
        node {
            label 'maven'
        }    
    }
    environment {
        PATH = "/opt/apache-maven-3.9.9/bin:$PATH"
    }
    stages {
        stage("build") {
            steps {
                //git branch: 'main', url: 'https://github.com/pranvi13/tweet-trend-for-Project2.git'
                echo "build started"
                sh 'mvn clean deploy'
                echo "build completed"
            }
        }    
            stage('SonarQube analysis') {
            environment {
            scannerHome = tool 'fqts-sonar-scanner'
                   }
            steps{
                withSonarQubeEnv('fqts-sonar-server') { 
                sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        
        }
    }
}
