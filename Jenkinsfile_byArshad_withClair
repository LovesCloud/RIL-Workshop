
properties([[$class: 'JiraProjectProperty', siteName: 'https://lovescloud.atlassian.net/'], pipelineTriggers([githubPush()])])
node {
   def mvnHome
   def scannerHome
   stage('SCM CheckOut') {
      cleanWs disableDeferredWipeout: true, notFailBuild: true
      git branch: 'develop', url: 'https://github.com/LovesCloud/RIL-Workshop.git'           
      mvnHome = tool 'M3'
     // def commit = sh(returnStdout: true, script: 'git log -1 --pretty=%B | cat')
      //def matcher = commit =~ ([a-zA-Z][a-zA-Z0-9_]+-[1-9][0-9]*)([^.]|\.[^0-9]|\.$|$)
      //print commit
      //print matcher
      //test
      scannerHome = tool 'sonar_scanner';
   }

   stage('Maven Build') {
      sh "'${mvnHome}/bin/mvn' -Dmaven.test.failure.ignore clean package"
        
   }
   
 stage('Test-JUnit') {
      sh "'${mvnHome}/bin/mvn' test surefire-report:report"
   }
   
  stage('Sonar Code Analysis') {
      withSonarQubeEnv('SonarQube') {
         //testing sonar
        sh "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectName=RIL-Workshop -Dsonar.projectKey=RILW -Dsonar.sources=src -Dsonar.java.binaries=target/"
      }
   }


   stage('Docker Build') {
      sh"""#!/bin/bash
         docker build . -t crud-mysql-vuejs:${BUILD_NUMBER}
      """
   }

   stage('Nexus-Push') {
      
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
   
     stage("Docker Scan"){
      sshagent(['jenkins']) {
       sh"""#!/bin/bash
       CLAIR_ADDR=http://35.202.31.140:6060 CLAIR_OUTPUT=Low CLAIR_THRESHOLD=10  JSON_OUTPUT=true klar lovescloud/crud-mysql-vuejs:${BUILD_NUMBER} | jq '.' | cat > crud-mysql-vuejs:${BUILD_NUMBER}.json
          """
    }
    sh label: '', script: 'sudo cp /var/lib/jenkins/workspace/RILW/crud-mysql-vuejs\\:${BUILD_NUMBER}.json /var/www/html/'
    }
   

   stage('Deploy') {
      sh label: '', script: '''sed -i \'s/IMAGE/image: lovescloud\\/crud-mysql-vuejs:\'${BUILD_NUMBER}\'/\' docker-compose.yaml
'''
      sh"""#!/bin/bash
      cat docker-compose.yaml
      kompose convert
      ls -las
      sudo mkdir ${BUILD_NUMBER}-kompose/
      ls -las ${BUILD_NUMBER}-kompose/
      sudo chown jenkins:jenkins ${BUILD_NUMBER}-kompose/
      sudo mv crud-mysql-vuejs-* ${BUILD_NUMBER}-kompose/
      sudo mv hk-mysql-* ${BUILD_NUMBER}-kompose/
      cd ${BUILD_NUMBER}-kompose/
      for f in * ; do mv -- "\$f" "${BUILD_NUMBER}_\$f" ; done
      cd ..
      kubectl apply -f ${BUILD_NUMBER}-kompose/
      sleep 10
      """
   }
   
stage('Cleanup') {
      
      cleanWs disableDeferredWipeout: true, notFailBuild: true
      emailext body: " ${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}",subject: "Jenkins Build ${currentBuild.currentResult}: Job ${env.JOB_NAME}", to: 'rajnikhattarrsinha@loves.cloud,rajnikhattarrsinha@gmail.com'

   }
   
  
  }
