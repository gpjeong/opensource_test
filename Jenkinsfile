pipeline {
    agent any

    environment {
        IMAGE_NAME = "myapp"
        CONTAINER_NAME = "test"
    }

    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    echo 'Build Docker Image Start'

                    // 기존에 실행 중인 컨테이너가 있다면 정지 후 삭제
                    sh "docker ps -q --filter name=${CONTAINER_NAME} | xargs -r docker stop"
                    sh "docker ps -a -q --filter name=${CONTAINER_NAME} | xargs -r docker rm"

                    // 기존에 사용하고 있던 이미지 삭제
                    sh "docker images --filter reference=${CONTAINER_NAME} --format '{{.ID}}' | xargs -r docker rmi -f"

                    // 코드를 빌드하고 Docker 이미지를 생성
                    def shortCommit = "${GIT_COMMIT}".substring(0, 7)
                    def imageNameWithTag = "${IMAGE_NAME}:${shortCommit}"
                    app =  docker.build(imageNameWithTag)

                    // 도커 이미지 중 <none> 태그 제거
                    def danglingImages = sh(script: 'docker images -q -f dangling=true', returnStdout: true).trim()
                    if (danglingImages) {
                        sh "docker rmi ${danglingImages}"
                    } else {
                        echo 'No dangling images to remove.'
                    }

                    echo 'Build Docker Image End'
                }
            }
        }

        stage('Deploy to Server') {
            steps {
                script {
                    echo 'Deploy to Server Start'
                    // 새로운 Cotainer 실행
                    def shortCommit = "${GIT_COMMIT}".substring(0, 7)
                    def imageNameWithTag = "${IMAGE_NAME}:${shortCommit}"
                    sh "docker run -it -d -p 8000:8000 --name ${CONTAINER_NAME} ${imageNameWithTag}"
                    echo 'Deploy to Server End'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    echo 'Push Docker Image Start'
                    def shortCommit = "${GIT_COMMIT}".substring(0, 7)
                    registry = "${HARBOR_PROTOCOL}://${HARBOR_IP}"
                    docker.withRegistry(registry, 'harborid') {
                        app.push("${shortCommit}")
                    echo 'Push Docker Image End'
                    }
                }
            }
        }
    }
}
