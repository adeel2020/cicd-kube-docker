pipeline {
    
	agent any
    /*	
	tools {
        maven "maven3"
	
    }*/
	
    environment {
        
	KUBECONFIG = '/home/ubuntu/.kube/config'
        registry = "adeel2020/vproappdock"
        registryCredential = "dockerhub"
    }
	
    stages{
        
        stage('BUILD'){
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
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
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
	}
        stage('CODE ANALYSIS with SONARQUBE') {
          
		/*  environment {
             scannerHome = tool 'sonarscanner4'
          }*/

          steps {
            withSonarQubeEnv('sonar-pro') {
               sh '''mvn sonar:sonar -Dsonar.login=8ef0e3948b8740e7ab7d11206951353bedf507fc \
                   -Dsonar.host.url=https://sonarcloud.io \
                   -Dsonar.projectKey=adeel2020_vprofile-repo \
                   -Dsonar.organization=adeel2020 \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
            }

            timeout(time: 10, unit: 'MINUTES') {
               waitForQualityGate abortPipeline: true
            }
          }
        }

        stage ('Build app Image') {
            steps {
                script {
                    dockerImage = docker.build registry + ":V$BUILD_NUMBER"
                }
            }
        }
        stage ('Upload Image'){
            steps {
                script{
                    docker.withRegistry('',registryCredential){
                        dockerImage.push("V$BUILD_NUMBER")
                        dockerImage.push('latest')
       		    }
                }
            }
        }
        
        stage ('Remove Unused Docker Image'){
            steps{
                sh "docker rmi $registry:V$BUILD_NUMBER"
            }
        }

        stage('Kubernetes Deploy') {
          agent {
                label 'KOPS' 
          }
    	    steps{
		//	sh 'kubectl get pods'
           	    	sh 'helm upgrade --install --force vprofile-stack helm/vprofilecharts --set appimage=${registry}:V${BUILD_NUMBER} --namespace prod -v 20'
            }
        }
            
    }


}

