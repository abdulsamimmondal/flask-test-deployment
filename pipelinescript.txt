pipeline {
    agent any

    stages {
        stage("Git pull") {
            steps {
                git url: 'https://github.com/abdulsamimmondal/flask-test-deployment', branch: 'main'
                echo "Cloned"
            }
        }

        stage("Adding the k8s files") {
            steps {
                sh '''
                    cp root/test-flask-pipeline/k8s/* /var/lib/jenkins/test-pipeline/flask-test-deployment/k8s/
                '''
            }
        }

        stage("Docker build") {
            steps {
                sh 'docker build -t samimmondal/test-flask-pipeline .'
            }
        }

        stage("Docker Push") {
            steps {
                withDockerRegistry([credentialsId: 'dockerhub-cred', url: '']) {
                    sh 'docker push samimmondal/test-flask-pipeline'
                }
            }
        }

        stage("K8s") {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh '''
                        kubectl apply -f k8s/. --namespace=default
                    '''
                }
            }
        }
    }
}
