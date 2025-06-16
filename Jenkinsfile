pipeline {
    agent any

    environment {
        IMAGE_NAME   = "car-web"
        IMAGE_TAG    = "1.0"
        DOCKER_REPO  = "arunawsdevops/car-web"
    }

    stages {
        stage('clone-gitrepo') {
            steps {
                checkout scmGit(
                    branches: [[name: '*/main']],
                    extensions: [],
                    userRemoteConfigs: [[
                        credentialsId: 'git_cred',
                        url: 'https://github.com/arunawsdevops/Car-web.git'
                    ]]
                )
                sh 'ls -l'
            }
        }

        stage('docker-build') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('list-docker-images') {
            steps {
                sh 'docker images'
            }
        }

        stage('Docker-image-push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker_hub_cred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${DOCKER_REPO}:${IMAGE_TAG}
                        docker push ${DOCKER_REPO}:${IMAGE_TAG}
                    """
                }
            }
        }
    }
}
