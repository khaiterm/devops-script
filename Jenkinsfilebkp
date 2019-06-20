import jenkins.model.*;
import org.jfrog.*;
import org.jfrog.hudson.*;
import org.jfrog.hudson.util.Credentials;

node {
   def mvnHome
   stage('Source') { // for display purposes
      // Get some code from a GitHub repository
      git 'https://github.com/khaiterm/sample.git'
      // Get the Maven tool.
      // ** NOTE: This 'M3' Maven tool must be configured
      // **       in the global configuration.           
      mvnHome = tool 'm3'
      
   }
   stage('Build') {
      // Run the maven build
      withEnv(["MVN_HOME=$mvnHome"]) {
    
            sh '"$MVN_HOME/bin/mvn" -Dmaven.test.failure.ignore clean install -Dv=${BUILD_NUMBER}'

      }
   }
stage('Test') {
       withSonarQubeEnv('sonarQube') {
      if(env.BRANCH_NAME == 'master') {
			def scannerHome = tool 'Sonar Scanner';
			withSonarQubeEnv('SonarQube') {
				sh "${scannerHome}/bin/sonar-scanner"}
        }
 
    sh 'mvn sonar:sonar -Dsonar.projectKey=sample -Dsonar.host.url=http://ec2-3-217-156-220.compute-1.amazonaws.com:9000 -Dsonar.login=0eea125aac63547051a22f11baadd7871378c660 -Dsonar.projectVersion=helloworld-example-1.0.0.${BUILD_NUMBER}-SNAPSHOT.jar'
    
}

stage('Upload') {

def server = Artifactory.newServer url: 'http://ec2-3-217-156-220.compute-1.amazonaws.com:8081/artifactory/', username: 'group6', password: 'adminadmin';

def uploadSpec = """{
  "files": [
    {
      "pattern": "/var/jenkins_home/workspace/sample/target/helloworld-example-1.0.0.${BUILD_NUMBER}-SNAPSHOT.jar",
      "target": "Jenkins-integration/${BUILD_NUMBER}/"
    }
 ]
}"""

def buildInfo = Artifactory.newBuildInfo()
server.upload spec: uploadSpec, buildInfo: buildInfo
server.publishBuildInfo buildInfo 

server.upload spec: uploadSpec, failNoOp: true
}

} 
}
