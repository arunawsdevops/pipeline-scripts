pipeline {
    agent any

    parameters {
        string(name: 'Run_by', description: 'Who is running the pipeline?')
        string(name: 'IMAGE_TAG', defaultValue: '1.0', description: 'Tag for the Docker image')
        string(name: 'IMAGE_NAME', defaultValue: 'car-web', description: 'Name for the Docker image')
    }

    environment {
        DOCKER_REPO = "arunawsdevops"
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
                sh "docker build -t ${params.IMAGE_NAME}:${params.IMAGE_TAG} ."
                echo "Build triggered by: ${params.Run_by}"
            }
        }

        stage('list-docker-images') {
            steps {
                sh 'docker images'
            }
        }

        stage('dockerhub-push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker_hub_cred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker tag ${params.IMAGE_NAME}:${params.IMAGE_TAG} ${DOCKER_REPO}/${params.IMAGE_NAME}:${params.IMAGE_TAG}
                        docker push ${DOCKER_REPO}/${params.IMAGE_NAME}:${params.IMAGE_TAG}
                        docker logout
                    """
                }
            }
        }

        stage('k8 deployment') {
            steps {
                withKubeConfig(
                    caCertificate: '', 
                    clusterName: '', 
                    contextName: '', 
                    credentialsId: 'eks_cred', 
                    namespace: '', 
                    restrictKubeConfigAccess: false, 
                    serverUrl: ''
                ) {
                    sh 'kubectl get nodes'
                    sh 'kubectl get ns'
                    sh 'kubectl apply -f car-deploy.yaml'
                    sh 'kubectl apply -f car-svc.yaml'
                    sh 'kubectl get pod'
                }
            }
        }
    }
}
