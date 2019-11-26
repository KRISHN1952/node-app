pipeline {
    agent any
    environment{
        DOCKER_TAG = getDockerTag()
    }
    stages{
        stage('Build Docker Image'){
            steps{
                sh "docker build . -t krishna1952/nodeapp:${DOCKER_TAG} "
            }
        }
        stage('DockerHub Push'){
            steps{
                withCredentials([string(credentialsId: 'DOCKER_HUB_CREDENTIALS', variable: 'DOCKER_HUB_CREDENTIALS')]) {
                    sh "docker login -u krishna1952 -p ${DOCKER_HUB_CREDENTIALS}"
                    sh "docker push krishna1952/nodeapp:${DOCKER_TAG}"
                }
            }
        }
        stage('Deploy to k8s'){
            steps{
                sh "chmod +x changeTag.sh"
                sh "./changeTag.sh ${DOCKER_TAG}"
                sshagent(['master']) {
                    sh "scp -o StrictHostKeyChecking=no services.yml node-app-pod.yml ssh toor@104.46.60.255:/home/toor/"
                    script{
                        try{
                            sh "ssh toor@104.46.60.255 kubectl apply -f ."
                        }catch(error){
                            sh "ssh toor@104.46.60.255 kubectl create -f ."
                        }
                    }
                }
            }
        }
    }
}

def getDockerTag(){
    def tag  = sh script: 'git rev-parse HEAD', returnStdout: true
    return tag
}
