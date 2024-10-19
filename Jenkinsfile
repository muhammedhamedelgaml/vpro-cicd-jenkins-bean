def COLOR_MAP = [
  'SUCCESS' : 'good',
  'FAILURE' : 'danger'
]
pipeline{
  agent any 
  tools {
    maven  "maven3"
    jdk  "jdk11"
   } 
   environment{
    SNAP_REPO = 'vprofile-snapshot'
    NEXUS_USER = 'admin'
    NEXUS_PASS = 'admin123'
    RELEASE_REPO = 'vprofile-release'
    CENTRAL_REPO = 'vpro-maven-central' 
    NEXUS_GRP_REPO = 'vpro-maven-group'
    NEXUSIP = 'localhost'
    NEXUSPORT =  '8081'
    NEXUS_LOGIN = 'nexuslogin' 
    //////////////////////////////
    ARTIFCAT_NAME = "vprofile-v${BUILD_ID}.war"
    AWS_S3_BUCKET = 'vpro-cicd-jenbean'
    AWS_EB_APP_NAME = 'vproapp' // application name on beanstalk
    AWS_EB_ENV      = 'vproapp-env'  // env name
    AWS_EP_APP_VERSION = "${BUILD_ID}"  // version name will be BUILD_ID of pipeline #!
    }

  stages{
      stage('Build'){
       steps{ 
         sh 'mvn -s settings.xml -DskipTests install'
       }
       post {
              success {
                echo 'Now Archiving...'
                archiveArtifacts artifacts: '**/target/*.war'
                }
            }
      }

// 	stage('UNIT TEST'){
//             steps {
//                 sh 'mvn test'
//             }
//         }

// 	stage('INTEGRATION TEST'){
//             steps {
//                 sh 'mvn verify -DskipUnitTests'
//             }
//         }
		
//         stage ('CODE ANALYSIS WITH CHECKSTYLE'){
//             steps {
//                 sh 'mvn checkstyle:checkstyle'
//             }
//             post {
//                 success {
//                     echo 'Generated Analysis Result'
//                 }
//             }
//         }

//         stage('CODE ANALYSIS with SONARQUBE') {
          
// 		  environment {
//              scannerHome = tool 'sonarscanner4'
//           }

//           steps {
//             withSonarQubeEnv('sonar-pro') {
//                sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
//                    -Dsonar.projectName=vprofile-repo \
//                    -Dsonar.projectVersion=1.0 \
//                    -Dsonar.sources=src/ \
//                    -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
//                    -Dsonar.junit.reportsPath=target/surefire-reports/ \
//                    -Dsonar.jacoco.reportsPath=target/jacoco.exec \
//                    -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
//             }

//             timeout(time: 10, unit: 'MINUTES') {
//                waitForQualityGate abortPipeline: true
//             }
//           }
//         }


  stage("uploadArtifact to nexus"){
   steps{
     nexusArtifactUploader(
        nexusVersion: 'nexus3',
        protocol: 'http',
        nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
        groupId: 'QA',
        version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
        repository: "${RELEASE_REPO}",
        credentialsId: "${NEXUS_LOGIN}",
        artifacts: [
           [ 
              artifactId: 'vproapp' ,
              classifier: '',
              file: 'target/vprofile-v2.war',
              type: 'war'
           ] 

        ]

     )
     
   }
 }


/////////////////////////////////////////////////////////////////////
//CD_stage

     stage('deploy to stage bean'){
      withAWS( credentials: 'beancreds' , region: 'us-east-1') {       // beancreds is save at jenkins credentials
     
      sh 'aws s3 cp ./target/vprofile-v2.war s3://$AWS_S3_BUCKET/$ARTIFCAT_NAME '  // upload  artifact to s3 
      sh 'aws elasticbeanstalk create-application-version --application-name $AWS_EP_APP_NAME  --version-label $AWS_EP_APP_VERSION --source-bundle S3Bucket=$AWS_S3_BUCKET,S3Key=$ARTIFACT_NAME' // create new application version in beanstalk 
      sh 'aws elasticbeanstalk update-enviroment --application-name= $aws_EP_APP_NAME --enviroment-name $AWS_EP_ENV --version-label $AWS_EP_APP_VERSION' // deploy this application version to beanstalk enviroment 

      } 
    }
      
     






  }
  
  //////////////////////////
  //slack INTEGRATION
  post {
        always  {
            echo 'slack notification.'
            slackSend channel: '#cicdjenkins',
            color:  COLOR_MAP[currentBuild.currentResult],
            message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n for more info visit : ${env.BUILD_URL} " 
        }
  }

}