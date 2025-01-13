def registry = 'https://fqts01.jfrog.io'
pipeline {
    agent {
        node {
            label 'maven'
        }
    }
    environment {
        PATH = "/opt/apache-maven-3.9.9/bin:$PATH"
        ARTIFACTORY_REPO = 'fqts01-docker-local'
        DOCKER_IMAGE_NAME = 'ttrend'
        DOCKER_TAG = '2.1.2'
        DOCKER_REGISTRY = 'https://fqts01.jfrog.io'
        ARTIFACTORY_CREDENTIALS = credentials('jfrog-creds')
    }
    stages {
        stage("build"){
            steps {
                 echo "----------- build started ----------"
                 sh 'mvn clean deploy'
                 echo "----------- build complted ----------"
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
        
         stage("Jar Publish") {
          steps {
            script {
                    echo '<--------------- Jar Publish Started --------------->'
                     def server = Artifactory.newServer url:registry+"/artifactory" ,  credentialsId:"jfrog-creds"
                     def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                     def uploadSpec = """{
                          "files": [
                            {
                              "pattern": "jarstaging/(*)",
                              "target": "fqts01-libs-release-local/{1}",
                              "flat": "false",
                              "props" : "${properties}",
                              "exclusions": [ "*.sha1", "*.md5"]
                            }
                         ]
                     }"""
                     def buildInfo = server.upload(uploadSpec)
                     buildInfo.env.collect()
                     server.publishBuildInfo(buildInfo)
                     echo '<--------------- Jar Publish Ended --------------->'  
            
                   }
              }   
           }
           stage('Build Docker Image') {
            steps {
                script {
                    sh """
                    docker build -t ${DOCKER_IMAGE_NAME}:${DOCKER_TAG} .
                    """
                }
            }
        }
        stage('Publish to Artifactory') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", 
                                                     usernameVariable: 'DOCKER_USER', 
                                                     passwordVariable: 'DOCKER_PASSWORD')]) 
													 
													 
                    sh """
                    docker login ${DOCKER_REGISTRY} -u ${DOCKER_USER} -p ${DOCKER_PASSWORD}
                    docker tag ${DOCKER_IMAGE_NAME}:${DOCKER_TAG} ${DOCKER_REGISTRY}/${DOCKER_IMAGE_NAME}:${DOCKER_TAG}
                    docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE_NAME}:${DOCKER_TAG}
                    """
                }
            }
        }
    }
}