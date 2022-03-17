pipeline {
	agent any
	//parameters {
	//	string (name: 'parameterTest', defaultValue: 'Paraaaaaa', description: 'parameter hello')
	//}
	/* pipeline 변수 설정 */
	//environment {
	//	// 로컬 리포지토리 사용
	//	// DockerUserName='wjdghks1057'
	//	// DockerUserName='192.168.143.123:5000'
	//	
	//	// 네트워크 환경의 도커 리포지토리 G1
	//	DockerUserName='192.168.143.151:5000'
	//	
	//	ProjectName='git-test'
	//	registryCredential = 'docker-hub'
	//}
	
	//tools {
	//	maven 'Maven 3.3.9'
	//	jdk 'jdk8'
	//}	
	
	options {
		retry(1)
		timestamps()
	}
	
	/* SCM 소스 checkout */
	stages{
		//stage('Initialize') {
        //    steps{
        //        echo "M2_HOME = /opt/maven"
		//		echo "PATH = ${M2_HOME}/bin:${PATH}"
		//		echo "PATH+EXTRA=/usr/local/bin"
        //    }
		//	post {
		//		failure {
		//			script { env.FAILURE_STAGE = 'Initialize' }
		//		}
		//	}
        //}		
	
		//stage('Checkout') {
		//	steps {
		//		echo "git Checkout stage..."	
		//	}
		//	post {
		//		failure {
		//			script { env.FAILURE_STAGE = 'Checkout' }
		//		}
		//	}
		//}
	
		stage('Maven Build') {
			agent {
				docker {
					image 'maven:3.3.9-jdk-8'
					// args '-v /root/.m2:/root/.m2'
				}
			}
			environment {
				JENKINS_RUNNING_IN_CONTAINER = true
			}
			steps {
				sh 'mvn clean install'
			}
			post {
				failure {
					script { env.FAILURE_STAGE = 'Maven Build' }
				}
			}
		}
		
		stage('Docker Build and Push') {
			agent {
				docker {
					image 'docker:latest'
				}
			}
			environment {
				DockerUserName='192.168.143.151:5000'
				ProjectName='git-test'
				registryCredential = 'docker-hub'
			}
			steps {
				sh "docker build -t $DockerUserName/$ProjectName:latest ."
				
				withDockerRegistry([credentialsId: registryCredential, url: ""]) {
					sh "docker tag $DockerUserName/$ProjectName:latest $DockerUserName/$ProjectName:$BUILD_NUMBER"
					sh "docker push $DockerUserName/$ProjectName:$BUILD_NUMBER"
					sh "docker push $DockerUserName/$ProjectName:latest"
				}
			}
			post {
				always { 
					// sh "docker logout"
					
					sh "echo Docker image Clean..."
					sh "docker rmi $DockerUserName/$ProjectName:$BUILD_NUMBER"
					sh "docker rmi $DockerUserName/$ProjectName:latest"
				}
				failure {
					script { env.FAILURE_STAGE = 'Docker push' }
				}
			}
		}
		
		stage('Deploy to k8s') {
			agent any
			steps {
				kubernetesDeploy(
					// kubeconfigId: 'Kubeconfig',
					kubeconfigId: 'kubeG1',
                    configs: 'deployment.yml',
					// yml 파일의 $var와 ${var}를 젠킨스 environment로 대체한다
                    enableConfigSubstitution: true
                )
			}
			post {
				failure {
					script { env.FAILURE_STAGE = 'Deploy to k8s' }
				}
			}
		}
	}
	
	 post {     
		success {
			slackSend tokenCredentialId: 'slackJenkinsId', color: "good", message: "${JOB_NAME} - #${BUILD_NUMBER} succeeeded (<${env.BUILD_URL}|Open>)"	
        }
        unstable {
			slackSend tokenCredentialId: 'slackJenkinsId', color: "warning", message: "${JOB_NAME} - #${BUILD_NUMBER} unstable (<${env.BUILD_URL}|Open>)"
        }
        failure {
			slackSend tokenCredentialId: 'slackJenkinsId', color: "danger", message: "${JOB_NAME} - #${BUILD_NUMBER} failed (<${env.BUILD_URL}|Open>)"
        }
        changed {
            echo "Things were different before..."
        }
    }
}