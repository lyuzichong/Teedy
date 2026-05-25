pipeline {
    agent any

    environment {
        JAVA_HOME = '/Users/lvzichong/Library/Java/JavaVirtualMachines/ms-25.0.0/Contents/Home'
        PATH = "$JAVA_HOME/bin:/opt/homebrew/bin:/usr/local/bin:/Users/lvzichong/.nvm/versions/node/v24.14.0/bin:$PATH"
        DOCKER_IMAGE = 'lv05327/teedy'
        DOCKER_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Package') {
            steps {
                sh 'mvn -B -DskipTests clean package'
            }
        }

        stage('Building image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
            }
        }

        stage('Upload image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'teedy', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "\$DOCKER_PASS" | docker login -u "\$DOCKER_USER" --password-stdin
                        for i in 1 2 3; do
                            docker push ${DOCKER_IMAGE}:${DOCKER_TAG} && break || sleep 10
                        done
                        docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest
                        for i in 1 2 3; do
                            docker push ${DOCKER_IMAGE}:latest && break || sleep 10
                        done
                    """
                }
            }
        }

        stage('Run containers') {
            steps {
                script {
                    def ports = [8082, 8083, 8084]
                    for (port in ports) {
                        def containerName = "teedy-container-${port}"
                        sh "docker stop ${containerName} || true"
                        sh "docker rm ${containerName} || true"
                        sh "docker run --name ${containerName} -d -p ${port}:8080 ${DOCKER_IMAGE}:${DOCKER_TAG}"
                    }
                }
            }
        }
    }
}
