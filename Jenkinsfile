pipeline {
    agent any

    stages {
        stage("Set Variable") {
            steps {
                script {
                    IMAGE_NAME = "d106"
                    IMAGE_STORAGE = "ohdoseok"
                    IMAGE_STORAGE_CREDENTIAL = "pkqwe37"
                    SSH_CONNECTION = "ubuntu@j7d106.p.ssafy.io"
                    SSH_CONNECTION_CREDENTIAL = "pkqwe37"
                }
            }
        }

        stage("Clean Build Test") {
            steps {
                sh "./gradlew clean build test"
            }
        }

        stage("Build Container Image") {
            steps {
                script {
                    image = docker.build("${IMAGE_STORAGE}/${IMAGE_NAME}")
                }
            }
        }

        stage("Push Container Image") {
            steps {
                script {
                    docker.withRegistry("https://${IMAGE_STORAGE}", IMAGE_STORAGE_CREDENTIAL) {
                        image.push("${env.BUILD_NUMBER}")
                        image.push("latest")
                        image
                    }
                }
            }
        }

        stage("Server Run") {
            steps {
                sshagent([SSH_CONNECTION_CREDENTIAL]) {
                    // 최신 컨테이너 삭제
                    sh "ssh -o StrictHostKeyChecking=no ${SSH_CONNECTION} 'docker rm -f ${IMAGE_NAME}'"
                    // 최신 이미지 삭제
                    sh "ssh -o StrictHostKeyChecking=no ${SSH_CONNECTION} 'docker rmi -f ${IMAGE_STORAGE}/${IMAGE_NAME}:latest'"
                    // 최신 이미지 PULL
                    sh "ssh -o StrictHostKeyChecking=no ${SSH_CONNECTION} 'docker pull ${IMAGE_STORAGE}/${IMAGE_NAME}:latest'"
                    // 이미지 확인
                    sh "ssh -o StrictHostKeyChecking=no ${SSH_CONNECTION} 'docker images'"
                    // 최신 이미지 RUN
                    sh "ssh -o StrictHostKeyChecking=no ${SSH_CONNECTION} 'docker run -d --name ${IMAGE_NAME} -p 8080:8080 ${IMAGE_STORAGE}/${IMAGE_NAME}:latest'"
                    // 컨테이너 확인
                    sh "ssh -o StrictHostKeyChecking=no ${SSH_CONNECTION} 'docker ps'"
                }
            }
        }
    }

}
