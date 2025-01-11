pipeline {
    agent {
        node {
            label 'maven'
        }
    }

    stages {
        stage('Clone-Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Prashant-dai-pune/tweet-trend-for-Project2.git'
            }
        }
    }
}