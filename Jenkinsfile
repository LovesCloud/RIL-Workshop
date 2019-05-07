node {
   def mvnHome
   def scannerHome
   stage('Prepare') { // for display purposes
      // Get some code from a GitHub repository
      git 'https://github.com/LovesCloud/RIL-Workshop.git'           
      mvnHome = tool 'M3'
      scannerHome = tool 'sonar_scanner';
   }

   stage('Build') {
      // Run the maven build
      if (isUnix()) {
         sh "'${mvnHome}/bin/mvn' -Dmaven.test.failure.ignore clean package"
        
      } else {
         bat(/"${mvnHome}\bin\mvn" -Dmaven.test.failure.ignore clean package/)
      }
   }
   
   stage('Test-JUnit') {
      // Run the maven build
      if (isUnix()) {
         sh "'${mvnHome}/bin/mvn' test surefire-report:report"
        
      } else {
         bat(/"${mvnHome}\bin\mvn" test/)
      }
   }
   
   stage('Sonar') {
      withSonarQubeEnv('SonarQube') {
        sh "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectName=RIL-W -Dsonar.projectKey=RIL-W -Dsonar.sources=src -Dsonar.java.binaries=target/"
      }
   }
}
