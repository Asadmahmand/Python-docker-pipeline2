pipeline {
    environment {
        registry = "asadmahmand/jenkins-docker-pipeline" //To push an image to Docker Hub, you must first name your local image using your Docker Hub username and the repository name that you created through Docker Hub on the web.
        registryCredential = 'docker-hub-login'
        dockerImage = ''
    }
    agent any
    stages {
        stage('checkout') {
            steps {
                git 'https://github.com/kss7/SimpleFlaskUI.git'
            }
        }

        stage('Stop previous running container') {
            steps {
                sh returnStatus: true, script: 'sudo docker stop $(docker ps -a | grep ${JOB_NAME} | awk \'{print $1}\')'
                sh returnStatus: true, script: 'sudo docker rmi $(docker images | grep ${registry} | awk \'{print $3}\') --force' //this will delete all images
                sh returnStatus: true, script: 'sudo docker rm ${JOB_NAME}'
            }
        }

        stage('Build Image') {
            steps {
                script {
                    img = registry + ":${env.BUILD_ID}"
                    println ("${img}")
                    dockerImage = docker.build("${img}")
                }
            }
        }

        stage('Test - Run Docker Container on Jenkins node') {
            steps {
                sh label: '', script: "docker run -d --name ${JOB_NAME} -p 5000:5000 ${img}"
            }
        }

        stage('Push To DockerHub') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', registryCredential) {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('running in staging') {
            steps {
                script {
                    def stopcontainer = "sudo docker stop ${JOB_NAME}"
                    def delcontName = "sudo docker rm ${JOB_NAME}"
                    def delimages = 'sudo docker image prune -a --force'
                    def drun = "sudo docker run -d --name ${JOB_NAME} -p 5000:5000 ${img}"
                    println "${drun}"
                    sshagent(['AWS-Wordpress-Server']) {
                        sh returnStatus: true, script: "ssh -o StrictHostKeyChecking=no ubuntu@3.108.234.74 ${stopcontainer}"
                        sh returnStatus: true, script: "ssh -o StrictHostKeyChecking=no ubuntu@3.108.234.74 ${delcontName}"
                        sh returnStatus: true, script: "ssh -o StrictHostKeyChecking=no ubuntu@3.108.234.74 ${delimages}"
                        sh "ssh -o StrictHostKeyChecking=no ubuntu@3.108.234.74 ${drun}"
                    }
                }
            }
        }
    }
}
