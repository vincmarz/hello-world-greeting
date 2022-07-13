node('docker') {
  stage('Poll') {
    checkout scm
  }
  stage('Build & Unit test'){
    sh 'mvn clean verify -DskipITs=true';
    junit '**/target/surefire-reports/TEST-*.xml'
    archive 'target/*.jar'
  }
  stage('Static Code Analysis'){
    withSonarQubeEnv('Default SonarQube Server') {
    sh 'mvn clean verify sonar:sonar -Dsonar.projectName=example-project -Dsonar.projectKey=example-project -Dsonar.projectVersion=$BUILD_NUMBER';
     }
    }
    stage ('Integration Test'){
      sh 'mvn clean verify -Dsurefire.skip=true';
      junit '**/target/failsafe-reports/TEST-*.xml'
      archive 'target/*.jar'
    }
    stage ('Publish'){
      
      /* def server = Artifactory.server 'Artifactory_JFrog_server'
      def uploadSpec = """{
        "files": [
           {
             "pattern": "target/hello-0.0.1.war",
             "target": "http://192.168.122.6:8082/artifactory/example-project/${BUILD_NUMBER}/",
             "props": "Integration-Tested=Yes;Performance-Tested=No"
            }
        ]
     }"""
     server.upload(uploadSpec)
     */
     rtUpload (
        serverId: 'Artifactory_JFrog_server',
        spec: '''{
          "files": [
            {
              "pattern": "target/hello-0.0.1.war",
              "target": "example-project/${BUILD_NUMBER}/",
              "props": "Integration-Tested=Yes;Performance-Tested=No"
            }
         ]
        }''',
        // Build name and build number for the build-info:
        //buildName: 'holyFrog',
        //buildNumber: '42',
        // Optional - Only if this build is associated with a project in Artifactory, set the project key as follows.
        project: 'example-project',
        // You also have the option of customising the build-info module name:
        //module: 'my-custom-build-info-module-name',
        //specPath: 'path/to/spec/relative/to/workspace/spec.json'
     )
   }
   stash includes:
     'target/hello-0.0.1.war,src/pt/Hello_World_Test_Plan.jmx',
     name: 'binary'
}
node('docker_pt') {
  stage ('Start Tomcat'){
    sh '''cd /home/jenkins/tomcat/bin 
          ./startup.sh''';
  }
  stage ('Deploy '){
    unstash 'binary'
    sh 'cp target/hello-0.0.1.war /home/jenkins/tomcat/webapps/';
  }
  stage ('Performance Testing'){
    sh '''cd /opt/jmeter/bin/
          ./jmeter.sh -n -t $WORKSPACE/src/pt/Hello_World_Test_Plan.jmx -l
          $WORKSPACE/test_report.jtl''';
    step([$class: 'ArtifactArchiver', artifacts: '**/*.jtl'])
  }
  stage ('Promote build in Artifactory'){
    withCredentials([usernameColonPassword(credentialsId:
      'artifactory-account', variable: 'credentials')]) {
         sh 'curl -u${credentials} -X PUT
         "http://192.:8081/artifactory/api/storage/example-project/${BUILD_NUMBER}/hello-0.0.1.war?properties=Performance-Tested=Yes"';
      }
  }
}  
