pipeline {
	agent any
	//parameters {
	//	string (name: 'parameterTest', defaultValue: 'Paraaaaaa', description: 'parameter hello')
	//}
	/* pipeline 변수 설정 */
	environment {
		// 로컬 리포지토리 사용
		// DockerUserName='wjdghks1057'
		// DockerUserName='192.168.143.123:5000'
		
		// 네트워크 환경의 도커 리포지토리 G1
		DockerUserName='192.168.143.151:5000'
		
		ProjectName='git-test'
		registryCredential = 'docker-hub'
		dbId = 'db_local_hello'
		app = ''
		
		PROJECT_ID = 'main-tine-344101'
        CLUSTER_NAME = 'gcp-kube-cluster-1'
        LOCATION = 'us-central1-c'
        CREDENTIALS_ID = 'gcp-k8s-project'
	}
	
	tools {
		maven 'Maven 3.3.9'
		jdk 'jdk8'
	}	
	
	options {
		retry(1)
	}
	
	/* SCM 소스 checkout */
	stages{
		stage('Initialize') {
            steps{
                echo "M2_HOME = /opt/maven"
				echo "PATH = ${M2_HOME}/bin:${PATH}"
				echo "PATH+EXTRA=/usr/local/bin"
            }
			post {
				failure {
					script { env.FAILURE_STAGE = 'Initialize' }
				}
			}
        }		
	
		stage('Checkout') {
			steps {
				echo "git Checkout stage..."	
			}
			post {
				failure {
					script { env.FAILURE_STAGE = 'Checkout' }
				}
			}
		}
	
		stage('Maven Build') {
			steps {
				sh 'mvn clean install'
			}
			post {
				failure {
					script { env.FAILURE_STAGE = 'Maven Build' }
				}
			}
		}
		
		stage('Docker Build image') {
			steps {
				sh 'docker build -t $DockerUserName/$ProjectName:latest .'				
			}
			post {
				failure {
					script { env.FAILURE_STAGE = 'Docker Build image' }
				}
			}
		}
		
		stage('Docker push') {	
			//input {
			//	message "Docker push input Test, continue?"
			//	ok "okkkkkkkk"
			//}
			
			steps {
				// withDockerRegistry([credentialsId: "", url: "192.168.143.123:5000"]) {
				withDockerRegistry([credentialsId: registryCredential, url: ""]) {
					sh 'docker tag $DockerUserName/$ProjectName:latest $DockerUserName/$ProjectName:$BUILD_NUMBER'
					sh 'docker push $DockerUserName/$ProjectName:$BUILD_NUMBER'
					sh 'docker push $DockerUserName/$ProjectName:latest'
				}
			}
			post {
				failure {
					script { env.FAILURE_STAGE = 'Docker push' }
				}
			}
		}
		
		stage('Deploy to k8s') {
			steps {
				kubernetesDeploy(
					// kubeconfigId: 'Kubeconfig',
					kubeconfigId: 'kubeG1',					
                    configs: 'deployment.yml',
                    enableConfigSubstitution: true
                )
			}
			post {
				failure {
					script { env.FAILURE_STAGE = 'Deploy to k8s' }
				}
			}
		}
		
		//stage('Deploy to GKE') {
		//	steps {
        //        step([
		//		$class: 'KubernetesEngineBuilder',
		//		projectId: env.PROJECT_ID,
		//		clusterName: env.CLUSTER_NAME,
		//		location: env.LOCATION,
		//		manifestPattern: 'gke-deployment.yaml',
		//		credentialsId: env.CREDENTIALS_ID,
		//		verifyDeployments: true])
		//	}
		//	post {
		//		failure {
		//			script { env.FAILURE_STAGE = 'Deploy to k8s' }
		//		}
		//	}
		//}
		
		//stage('Docker image Clean') {
		//	steps {
		//		sh 'docker rmi $DockerUserName/$ProjectName:$BUILD_NUMBER'
		//		sh 'docker rmi $DockerUserName/$ProjectName:latest'
		//	}
		//	post {
		//		failure {
		//			script { env.FAILURE_STAGE = 'Docker image Clean'}
		//		}
		//	}
		//}
	}
	
	 post { 
        always { 
            sh "docker logout"
			
			sh "echo Docker image Clean..."
			sh 'docker rmi $DockerUserName/$ProjectName:$BUILD_NUMBER'
			sh 'docker rmi $DockerUserName/$ProjectName:latest'
        }
		success {
			slackSend tokenCredentialId: 'slackJenkinsId', color: "good", message: "${JOB_NAME} - #${BUILD_NUMBER} succeeeded (<${env.BUILD_URL}|Open>) REVISION: $SVN_REVISION"	
        }
        unstable {
			slackSend tokenCredentialId: 'slackJenkinsId', color: "warning", message: "${JOB_NAME} - #${BUILD_NUMBER} unstable (<${env.BUILD_URL}|Open>) REVISION: $SVN_REVISION"
        }
        failure {
			slackSend tokenCredentialId: 'slackJenkinsId', color: "danger", message: "${JOB_NAME} - #${BUILD_NUMBER} failed (<${env.BUILD_URL}|Open>) REVISION: $SVN_REVISION"
        }
        changed {
            echo "Things were different before..."
        }
    }
}