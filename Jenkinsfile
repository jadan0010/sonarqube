// Jenkins file will have what is required for creating pipelie and pipeline stage and installation of tools like Sonarqube etc
//Jenkins work with Pom file. Let say we do the build phase, it will ceck pom file and download required tools as per Pom file and do the build
pipeline {
    agent any
    parameters {
//this parameters will be asked during pipelien build
    string(name: 'versionid', defaultValue: '1.0', description: 'Provide version ID')
    }
//tools to be installed for doing the build by jenkins. by default it will take tool from local; if the tools are not there in local, it will install tool in jenkis local 
//workspace and use the tools to do the build. As i already have these tools in local, i don't need to install in Jenkins workspace
    //tools { 
       // maven 'Maven' 
        //jdk 'JAVA_17' 
    //}
//Mvn –B DskiptTests clean package -> it will skip the test phase and do clean and do the package from java main folder file location in github; 
// it will check pom file to do the build
    stages {
        stage('Build') {
            steps {
                bat 'mvn -B -DskipTests clean package'
            }
        }
//. We are asking maven to check the test folder of github repository where the junit tests(unit test cases) are there and post the results to target and .xml file
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
// sonarscanner tool to be used jenkins. similar like mavern and java. this will install sonarscanner in jenkins. This will actual scan the code.
//sonarqube is to show output by validating code against rules and vulnerablitites defined in sonarqube
        stage('SonarQube analysis') {
          environment {
            SCANNER_HOME = tool 'Sonar-scanner'
            }
//bat > is windows powershell script similar linux script
//sonarqube id, token (so that jenkins can acces Sonarqube) and details are given. so that scan details will be posted to sonarqube
//projeckey and project name= it is project name in sonarqube u have crated; dsonar.source= source file locatin in github
//exclusion: exclude from scanning; take the build no and version id from previous input given
          steps {
            withSonarQubeEnv(credentialsId: 'sonartoken', installationName: 'localsonar') {
            bat "${SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectKey=jenkins-test -Dsonar.projectName=jenkins-test -Dsonar.sources=src/ -Dsonar.java.binaries=target/classes/ -Dsonar.exclusions=src/test/java/****/*.java  -Dsonar.projectVersion=${BUILD_NUMBER}-${params.versionid}"
            }
            }
        }
//create stage as squality gate and add 10 sec wait time before u  do next step
	    
// qualitygate to check code with vulnerablity condtions of code. It is part of sonarqube
        stage('Squality Gate') {
          steps {
                sleep(10)  /* Added 10 sec sleep that was suggested in few places*/
//This script will wait for 2 min to check qualitygate status verification.if status is not ok, it wil print error message for quality gate failure
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
