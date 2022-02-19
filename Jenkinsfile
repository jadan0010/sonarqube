pipeline {
    agent any
    parameters {
//this parameters will be asked during pipelien build
    string(name: 'versionid', defaultValue: '1.0', description: 'Provide version ID')
    }
//tools to be installed for doing the build by jenkins. by default it will take tool from local
    //tools { 
       // maven 'Maven' 
        //jdk 'JAVA_17' 
    //}
    stages {
        stage('Build') {
            steps {
                bat 'mvn -B -DskipTests clean package'
            }
        }
        stage('Test') {
            steps {
                bat 'mvn test'
            }
			//test reports are stored in xml
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
		// sonarscanner tool to be used jenkins. similar like mavern and java. this will install sonarscanner in jenkins. This will actual scan the code and sonarqube is to show //output
        stage('SonarQube analysis') {
          environment {
            SCANNER_HOME = tool 'Sonar-scanner'
            }
		//sonarqube id and details are given. so that scan details will be posted to sonarqube
		//exclusion: exclude from scanning; source: files to scan; then give project name and key
          steps {
            withSonarQubeEnv(credentialsId: 'sonartoken', installationName: 'localsonar') {
            bat "${SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectKey=jenkins-test -Dsonar.projectName=jenkins-test -Dsonar.sources=src/ -Dsonar.java.binaries=target/classes/ -Dsonar.exclusions=src/test/java/****/*.java  -Dsonar.projectVersion=${BUILD_NUMBER}-${params.versionid}"
            }
            }
        }
		//default steps; 2 min to wait timeout for Quality gate check in sonarqube. quality to check code with vulnerablity condtion
        stage('Squality Gate') {
          steps {
                sleep(10)  /* Added 10 sec sleep that was suggested in few places*/
                script{
                    timeout(time: 2, unit: 'MINUTES') {
                        def qg = waitForQualityGate abortPipeline: true
                        if (qg.status != 'OK') {
                            echo "Status: ${qg.status}"
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }
                }
            }   
        }
        stage('Deploy') {
          steps {
            bat "echo Deployed version ${params.versionid} Successfully"
            }
        }
    }
}
