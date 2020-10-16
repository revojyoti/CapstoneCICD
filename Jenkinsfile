pipeline {
        agent any
        stages {

                stage('Linting the HTML file') {
                        steps {
                                sh 'tidy -q -e *.html'
                        }
                }

                stage('Build Docker Image') {
                        steps {
                                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub', usernameVariable: 'DOCKERUSER', passwordVariable: 'DOCKERPASS']]){
                                        sh '''
                                                docker build -t jyotirevo/capstone .
                                        '''
                                }
                        }
                }

                stage('Push Image To Docker Hub') {
                        steps {
                                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
                                        sh '''
                                                docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
                                                docker push jyotirevo/capstone
                                        '''
                                }
                        }
                }
		stage('Create Kube config file') {
			steps {
				withAWS(region:'us-west-2', credentials:'TestID') {
					sh '''
						kubectl config use-context arn:aws:eks:us-west-2:692013865514:cluster/CapKubeClust
					'''
				}
			}
		}

		stage('Blue Controller Container Deploy') {
			steps {
				withAWS(region:'us-west-2', credentials:'TestID') {
					sh '''
						kubectl apply -f ./BlueController.yaml
					'''
				}
			}
		}
		stage('Green Controller Container Deploy') {
                        steps {
                                withAWS(region:'us-west-2', credentials:'TestID') {
                                        sh '''
                                                kubectl apply -f ./GreenController.yaml
                                        '''
                                }
                        }
                }
		stage('Run the Service in Blue Container') {
			steps {
				withAWS(region:'us-west-2', credentials:'TestID') {
					sh '''
						kubectl apply -f ./BlueService.yaml
					'''
				}
			}
		}

		stage('If OK Continue to move the updated Service to GReen Conttroller') {
            steps {
                input "Move to green?"
            }
        }
		stage('Run the Service in Green Container') {
                        steps {
                                withAWS(region:'us-west-2', credentials:'TestID') {
                                        sh '''
                                                kubectl apply -f ./GreenService.yaml
                                        '''
                                }
                        }
                }

        }
}
