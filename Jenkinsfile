pipeline {
	agent any
	stages {
		
		stage('Linting the HTML file') {
			steps {
				sh 'tidy -q -e index.html'
				sh 'hadolint Dockerfile'
			}
		}
		
		stage('Build Docker Image') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'revojyoti', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh '''
						docker build -t <USername/Passord> .
					'''
				}
			}
		}

		stage('Push Image To Docker Hub') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'revojyoti', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh '''
						docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
						docker push <username/passord>
					'''
				}
			}
		}
		stage('Set current kubectl context') {
			steps {
				withAWS(region:'us-west-2', credentials:'TestID') {
					sh '''
						kubectl config use-context arn:aws:eks:us-west-2:368648265814:cluster/CapKubeClusterMst
					'''
				}
			}
		}

		stage('Deploy blue container') {
			steps {
				withAWS(region:'us-west-2', credentials:'TestID') {
					sh '''
						kubectl apply -f ./blue-controller.json
					'''
				}
			}
		}

		stage('Deploy green container') {
			steps {
				withAWS(region:'us-west-2', credentials:'TestID') {
					sh '''
						kubectl apply -f ./green-controller.json
					'''
				}
			}
		}

		stage('Create the service in the cluster, redirect to blue') {
			steps {
				withAWS(region:'us-west-2', credentials:'TestID') {
					sh '''
						kubectl apply -f ./blue-service.json
					'''
				}
			}
		}

		stage('Waiting for user approve') {
            steps {
                input "All Done ? Are we ready to redirect user traffic to green?"
            }
        }

		stage('Create the service in the cluster, redirect to green') {
			steps {
				withAWS(region:'us-west-2', credentials:'TestID') {
					sh '''
						kubectl apply -f ./green-service.json
					'''
				}
			}
		}

	}
}
