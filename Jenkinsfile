#!/usr/bin/env groovy

import jenkins.model.*;
import org.jfrog.*;
import org.jfrog.hudson.*;
import org.jfrog.hudson.util.Credentials;

node {

  def mvnHome

  stage('Source') {
    notifyBuild('STARTED')
    // Get some code from a GitHub repository
    git url: 'https://github.com/khaiterm/example.git', branch: 'develop'
    // Get the Maven tool.
    // ** NOTE: This 'M3' Maven tool must be configured
    // **       in the global configuration.
    mvnHome = tool 'm3'
  }

  stage('Build') {
    withEnv(["MVN_HOME=$mvnHome"]) {
      sh '"$MVN_HOME/bin/mvn" -Dmaven.test.failure.ignore clean install -Dv=${BUILD_NUMBER}'
    }
  }

  stage('Test') {
    withSonarQubeEnv('sonarQube') {
      if(env.BRANCH_NAME == 'develop') {
        def scannerHome = tool 'Sonar Scanner';
        withSonarQubeEnv('SonarQube') {
          sh "${scannerHome}/bin/sonar-scanner"}
        }
      sh 'mvn sonar:sonar -Dsonar.projectKey=sample -Dsonar.host.url=http://172.31.18.118:9000 -Dsonar.login=0eea125aac63547051a22f11baadd7871378c660 -Dsonar.projectVersion=helloworld-example-1.0.0.${BUILD_NUMBER}-SNAPSHOT.war'
    }
  }

  stage('Upload') {
    sh 'scp /var/jenkins_home/.m2/repository/com/example/helloworld/1.0.0.${BUILD_NUMBER}-SNAPSHOT/helloworld-1.0.0.${BUILD_NUMBER}-SNAPSHOT.war mrevita@172.31.18.118:/opt/sample-java-app/helloworld.war'
    def server = Artifactory.newServer url: 'http://172.31.18.118:8081/artifactory/', username: 'group6', password: 'adminadmin';
    def uploadSpec = """{
      "files": [
        {
          "pattern": "/var/jenkins_home/.m2/repository/com/example/helloworld/1.0.0.${BUILD_NUMBER}-SNAPSHOT/helloworld-1.0.0.${BUILD_NUMBER}-SNAPSHOT.war",
          "target": "Jenkins-integration/${BUILD_NUMBER}/"
        }
     ]
    }"""
    def buildInfo = Artifactory.newBuildInfo()
    server.upload spec: uploadSpec, buildInfo: buildInfo
    server.publishBuildInfo buildInfo
    server.upload spec: uploadSpec, failNoOp: true
  }

  notifyBuild(currentBuild.result)
  
  stage('Deploy') {
   
      sh 'ansible-playbook -i /root/ansible/hosts /root/ansible/startweb.yml'
  }

}

def notifyBuild(String buildStatus = 'STARTED') {
  buildStatus =  buildStatus ?: 'SUCCESSFUL'

  def colorName = 'RED'
  def colorCode = '#FF0000'
  def subject = "Job: ${env.JOB_NAME}\nBuild: #${env.BUILD_NUMBER}\nStatus: ${buildStatus}"
  def summary = "${subject}\nURL: ${env.BUILD_URL}"
  def details = """
  STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':



  Check console output at "${env.JOB_NAME} [${env.BUILD_NUMBER}]"

  """

  if (buildStatus == 'STARTED') {
    color = 'YELLOW'
    colorCode = '#FFFF00'
  } else if (buildStatus == 'SUCCESSFUL') {
    color = 'GREEN'
    colorCode = '#00FF00'
  } else {
    color = 'RED'
    colorCode = '#FF0000'
  }

  slackSend (color: colorCode, message: summary)
  
}
