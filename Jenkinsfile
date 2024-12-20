pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "ahmedhussein18/ivolve-image" // Replace with your DockerHub repo
        KUBE_DEPLOY_FILE = "deployment.yaml"
    }
    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/IbrahimAdell/Lab.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE} ."
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-access-token', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin"
                        sh "docker push ${DOCKER_IMAGE}"
                    }
                }
            }
        }
        stage('Ensure Deployment YAML Exists') {
            steps {
                script {
                    if (!fileExists(KUBE_DEPLOY_FILE)) {
                        sh "kubectl create deployment ivolve --image=${DOCKER_IMAGE} --replicas=3 --dry-run=client -o yaml > ${KUBE_DEPLOY_FILE}"
                    }
                }
            }
        }
        stage('Update Deployment YAML') {
            steps {
                script {
                    sh "sed -i 's|<IMAGE_PLACEHOLDER>|${DOCKER_IMAGE}|g' ${KUBE_DEPLOY_FILE}"
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-file-id', variable: 'KUBE_CONFIG')]) {
                    script {
                        sh '''
                        export KUBECONFIG=$KUBE_CONFIG
                        kubectl apply -f ${KUBE_DEPLOY_FILE}
                        '''
                    }
                }
            }
        }
    }
    post {
        always {
            echo "Pipeline completed!"
        }
        success {
            echo "Pipeline succeeded!"
        }
        failure {
            echo "Pipeline failed."
        }
    }
}
