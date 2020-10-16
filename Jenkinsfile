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
        }
}
