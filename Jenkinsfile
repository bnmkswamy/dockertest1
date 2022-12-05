pipeline {
    environment {
        registry = 'kumaraswamy11/swamydev'
        registryCredential = 'dockerhub_id'
        devcontext = 'dev-swarm'
        prodcontext = 'prod-swarm'
        devnode = '172.31.19.117'
        prodnode = '172.31.17.1'
        dockerImage = ''
    }
    agent any
    stages {
        stage('Building Dev Docker Branch') {
            when {
                expression {
                    return env.BRANCH_NAME != 'Swamy-Dev'
                }
            }
            steps {
                script {
                    dockerImage = docker.build registry + ":v$BUILD_NUMBER"
                }
            }
        }
        stage('Push Image To DockerHUB') {
            steps {
                script {
                    docker.withRegistry( '', registryCredential ) {
                        dockerImage.push()
                    }
                }
            }
        }
        stage('Cleaning up') {
            steps {
                sh "docker rmi $registry:v$BUILD_NUMBER"
            }
        }
        stage('Deploying to Dev Docker Swarm') {
            steps {
                sh "docker context use $devcontext"
                sh 'docker service rm testing1 || true'
                sh "docker service create --name testing1 -p 8100:80 $registry:v$BUILD_NUMBER"
            }
        }
        stage('Verifying The Deployment') {
            steps {
                sh 'curl http://$devnode:8100 || exit 1'
            }
        }
        stage('Change Context To Default') {
            steps {
                sh 'docker context use default'
            }
        }
    }
}
