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
}
