@NonCPS
 //def getCommitMsg() {
 //   def commit = sh(returnStdout: true, script: 'git log -1 --pretty=%B | cat')
 //   def matcher = (commit =~ '([a-zA-Z][a-zA-Z0-9_]+-[1-9][0-9]*)([^.]|\.[^0-9]|\.$|$)')
 //   print matcher
 //   return matcher ? matcher[0][1] : null
 //}


node {
   try {
      
   def mvnHome
   def scannerHome
   stage('Prepare') {
      cleanWs disableDeferredWipeout: true, notFailBuild: true
      git 'https://github.com/LovesCloud/RIL-Workshop.git'           
      mvnHome = tool 'M3'
      scannerHome = tool 'sonar_scanner';
   }

   stage('Build') {
      sh "'${mvnHome}/bin/mvn' -Dmaven.test.failure.ignore clean package"
        
   }
   
   stage('Test-JUnit') {
      sh "'${mvnHome}/bin/mvn' test surefire-report:report"
   }
   
   stage('Sonar') {
      withSonarQubeEnv('SonarQube') {
        sh "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectName=RIL-Workshop -Dsonar.projectKey=RILW -Dsonar.sources=src -Dsonar.java.binaries=target/"
      }
   }


   stage('Docker-Build') {
      sh"""#!/bin/bash
         docker build . -t crud-mysql-vuejs:${BUILD_NUMBER}
      """
   }

   stage('Docker-Push') {
      withDockerRegistry(credentialsId: 'nexus', url: 'http://nexus.loves.cloud:8083') {
          sh"""#!/bin/bash
             docker tag crud-mysql-vuejs:${BUILD_NUMBER} nexus.loves.cloud:8083/crud-mysql-vuejs:${BUILD_NUMBER}
             docker push nexus.loves.cloud:8083/crud-mysql-vuejs:${BUILD_NUMBER}
             docker tag  nexus.loves.cloud:8083/crud-mysql-vuejs:${BUILD_NUMBER} crud-mysql-vuejs:${BUILD_NUMBER}
          """
      }

      withDockerRegistry(credentialsId: 'dockerhub') {
         sh"""#!/bin/bash
             docker tag crud-mysql-vuejs:${BUILD_NUMBER} lovescloud/crud-mysql-vuejs:${BUILD_NUMBER}
             docker push lovescloud/crud-mysql-vuejs:${BUILD_NUMBER}
          """
      }

      sh"""#!/bin/bash
         docker rmi lovescloud/crud-mysql-vuejs:${BUILD_NUMBER}
      """
      
   }

   stage('Trigger-Deploy') {
      sh label: '', script: '''sed -i \'s/IMAGE/image: lovescloud\\/crud-mysql-vuejs:\'${BUILD_NUMBER}\'/\' docker-compose.yaml
'''
      sh"""#!/bin/bash
      cat docker-compose.yaml
      kompose convert
      sudo mkdir ${BUILD_NUMBER}-kompose/
      sudo chown jenkins:jenkins ${BUILD_NUMBER}-kompose/
      sudo mv crud-mysql-vuejs-* ${BUILD_NUMBER}-kompose/
      sudo mv hk-mysql-* ${BUILD_NUMBER}-kompose/
      cd ${BUILD_NUMBER}-kompose/
      for f in * ; do mv -- "\$f" "${BUILD_NUMBER}_\$f" ; done
      cd ..
      kubectl apply -f ${BUILD_NUMBER}-kompose/

      sleep 10
      """
      jiraNotifyIssue failOnError: false, site: 'LC-Jira'
   }

   stage('Cleanup') {
      //getCommitMsg()
      def commit = sh(returnStdout: true, script: 'git log -1 --pretty=%B | cat')
          def matcher = (commit =~ '([a-zA-Z][a-zA-Z0-9_]+-[1-9][0-9]*)([^.]|\.[^0-9]|\.$|$)')
    print matcher
          if (matcher)
          {
            //mail bcc: '', body: 'BUILD_URL', cc: '', from: '', replyTo: '', subject: matcher[0][1]+'Build was successfully', to: 'jira@example.com'
          }
      cleanWs disableDeferredWipeout: true, notFailBuild: true
   }
   } catch(Exception e) {
      print "Error"
      print e
      raise e
   }

}
