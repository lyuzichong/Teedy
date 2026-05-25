pipeline {
    agent any

    environment {
        JAVA_HOME = '/Users/lvzichong/Library/Java/JavaVirtualMachines/ms-25.0.0/Contents/Home'
        PATH = "$JAVA_HOME/bin:/opt/homebrew/bin:/usr/local/bin:$PATH"
        DOCKER_HUB_CREDENTIALS = 'dockerhub_credentials'
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
                script {
                    docker.build("${env.DOCKER_IMAGE}:${env.DOCKER_TAG}")
                }
            }
        }

        stage('Upload image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', DOCKER_HUB_CREDENTIALS) {
                        docker.image("${env.DOCKER_IMAGE}:${env.DOCKER_TAG}").push()
                        docker.image("${env.DOCKER_IMAGE}:${env.DOCKER_TAG}").push('latest')
                    }
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
                        docker.image("${env.DOCKER_IMAGE}:${env.DOCKER_TAG}").run(
                            "--name ${containerName} -d -p ${port}:8080"
                        )
                    }
                }
            }
        }
    }
}
