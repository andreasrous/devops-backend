pipeline {
    agent any

    options {
        buildDiscarder(logRotator(numToKeepStr: '30', artifactNumToKeepStr: '30'))
    }

    environment {
        DOCKER_TOKEN = credentials('docker-push-secret')
        DOCKER_USER = 'andreasrous'
        DOCKER_SERVER = 'ghcr.io'
        DOCKER_PREFIX = 'ghcr.io/andreasrous/devops-backend'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'git@github.com:andreasrous/devops-backend.git'
            }
        }

        stage('Test') {
            steps {
                sh './mvnw test'
            }
        }

        stage('Docker build and push') {
            steps {
                sh '''
                    HEAD_COMMIT=$(git rev-parse --short HEAD)
                    TAG=$HEAD_COMMIT-$BUILD_ID
                    docker build --rm -t $DOCKER_PREFIX:$TAG -t $DOCKER_PREFIX:latest  -f nonroot.Dockerfile .
                    echo $DOCKER_TOKEN | docker login $DOCKER_SERVER -u $DOCKER_USER --password-stdin
                    docker push $DOCKER_PREFIX --all-tags
                '''
            }
        }

        stage('Deploy Certificate Issuer') {
            steps {
                sh '''
                    kubectl apply -f k8s/cert/cert-issuer.yaml
                '''
            }
        }

        stage('Deploy PostgreSQL') {
            steps {
                sh '''
                    kubectl apply -f k8s/postgres/postgres-pvc.yaml
                    kubectl apply -f k8s/postgres/postgres-svc.yaml
                    kubectl apply -f k8s/postgres/postgres-deployment.yaml
                '''
            }
        }

        stage('Deploy Spring Application') {
            steps {
                sh '''
                    kubectl apply -f k8s/spring/spring-svc.yaml
                    kubectl apply -f k8s/spring/spring-deployment.yaml
                    kubectl apply -f k8s/spring/spring-ingress-tls.yaml
                '''
            }
        }

        stage('Deploy Vue.js Application') {
            steps {
                sh '''
                    kubectl apply -f k8s/vue/vue-svc.yaml
                    kubectl apply -f k8s/vue/vue-deployment.yaml
                    kubectl apply -f k8s/vue/vue-ingress-tls.yaml
                '''
            }
        }

        stage('Deploy to k8s') {
            steps {
                sh '''
                    HEAD_COMMIT=$(git rev-parse --short HEAD)
                    TAG=$HEAD_COMMIT-$BUILD_ID
                    # if we had multiple configurations in kubeconfig file, we should select the correct one
                    # kubectl config use-context devops
                    kubectl set image deployment/spring-deployment spring=$DOCKER_PREFIX:$TAG
                    kubectl rollout status deployment spring-deployment --watch --timeout=2m
                '''
            }
        }
    }
}
