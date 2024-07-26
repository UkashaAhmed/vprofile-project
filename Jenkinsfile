pipeline {
    
	agent any
/*	
	tools {
        maven "maven3"
    }
*/	
    environment {
      SNAP_REPO = 'vprofile-snapshot'
      NEXUS_USER = 'admin'
      NEXUS_PASS = 'admin123'
      RELEASE_REPO = 'vprofile-release'
      CENTRAL_REPO = 'vpro-maven-central'
      NEXUSIP = '172.31.29.164'
      NEXUSPORT = '8081'
      NEXUS_GRP_REPO = 'vpro-maven-group'
      NEXUS_LOGIN = 'nexuslogin'
      SONARSERVER = 'sonarserver'
      SONARSCANNER = 'sonarscanner'
    }
	
    stages{
        
        stage('BUILD'){
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }

	stage('UNIT TEST'){
            steps {
                sh 'mvn test'
            }
        }

	stage('INTEGRATION TEST'){
            steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }
		
        stage ('CODE ANALYSIS WITH CHECKSTYLE'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
          
        }

        stage('CODE ANALYSIS with SONARQUBE') {
          
		  environment {
             scannerHome = tool "${SONARSCANNER}"
          }

          steps {
            withSonarQubeEnv("${SONARSERVER}") {
               sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile-repo \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
            }

            // timeout(time: 10, unit: 'MINUTES') {
            //    waitForQualityGate abortPipeline: true
            // }
          }
        }

        stage("Publish to Nexus Repository Manager") {
            steps {
              nexusArtifactUploader(
                            nexusVersion: 'nexus3',
                            protocol: 'http',
                            nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
                            groupId: 'QA',
                            version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                            repository: "${RELEASE_REPO}",
                            credentialsId: "${NEXUS_LOGIN}",
                            artifacts: 
                                [artifactId: 'vproapp',
                                classifier: '',
                                file: 'target/vprofile-v2.war',
                                type: 'war'],
                                
                            
                        );
            }
        }


    }


}

