pipeline {
    agent any
    environment {
        FRONTEND_IMAGE_REPO = 'techstore-web'
        FRONTEND_IMAGE_NAME = 'v0.2'
        BACKEND_IMAGE_REPO = 'techstore-api'
        BACKEND_IMAGE_NAME = 'v0.2'
        AWS_REGION = 'ap-southeast-1'
        REGISTRY_URL = "117001856078.dkr.ecr.${AWS_REGION}.amazonaws.com"
        CLUSTER_NAME = "ecommerce-nextapp-cluster"
    }
    stages {
        stage("init") {
            steps {
                script {
                    echo 'Init git submodules ...'
                    sh "git submodule init"
                    sh "git submodule update"
                    sh "ls frontend/"
                    sh "ls backend/"
                }
            }
        }
        stage("Create eks infrastructure") {
            steps {
                script {
                    dir('./terraform/') {
                        sh "terraform init"
                        sh "terraform plan"
                        sh "terraform apply -auto-approve"
                    }
                }
            }
        }
        stage("build backend image") {
            steps {
                script {
                    
                    echo 'Building backend image ...'
                    sh "docker build -t ${BACKEND_IMAGE_REPO}:${BACKEND_IMAGE_NAME} ./backend"
                }
            }
        }
        stage("push image") {
            steps {
                script {
                    echo 'Pushing backend image to ECR ...'
                    withCredentials([usernamePassword(credentialsId: 'aws-root', usernameVariable: "AWS_ACCESS_KEY_ID", passwordVariable: "AWS_SECRET_ACCESS_KEY")]) {
                        sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${REGISTRY_URL}"
                        sh "docker tag ${BACKEND_IMAGE_REPO}:${BACKEND_IMAGE_NAME} ${REGISTRY_URL}/${BACKEND_IMAGE_REPO}:${BACKEND_IMAGE_NAME}"
                        sh "docker push ${REGISTRY_URL}/${BACKEND_IMAGE_REPO}:${BACKEND_IMAGE_NAME}"
                    }
                }
            }
        }
        stage("deploy backend ") {
            environment {
                AWS_ACCESS_KEY_ID = credentials('jenkins_aws_access_key_id')
                AWS_SECRET_ACCESS_KEY = credentials('jenkins-aws_secret_access_key')
            }
            steps {
                script {
                    echo "Deploying backend to eks ..."
                    def ECR_PWD = sh(script: "aws ecr get-login-password --region ${AWS_REGION}", returnStdout: true)
                    sh "aws eks --region ${AWS_REGION} update-kubeconfig --name ${CLUSTER_NAME}"
                    sh "kubectl apply -f kubernetes/secret.yaml"
                    sh "kubectl create secret docker-registry my-ecr-key \
                    --docker-server=${REGISTRY_URL} \
                    --docker-username=AWS \
                    --docker-password=${ECR_PWD}"
                    sh "kubectl apply -f kubernetes/configmap.yaml"
                    sh "kubectl apply -f kubernetes/mongodb-vol.yaml"
                    sh "kubectl apply -f kubernetes/mongodb.yaml"
                    sh "kubectl apply -f kubernetes/backend.yaml"
                }
            }
        }
        stage("build frontend image") {
            steps {
                script {
                    echo 'Building frontend image ...'
                    sh "docker build -t ${FRONTEND_IMAGE_REPO}:${FRONTEND_IMAGE_NAME} ./frontend"
                }
            }
        }
        stage("push frontend image") {
            steps {
                script {
                    echo 'Pushing frontend image to ECR ...'
                    withCredentials([usernamePassword(credentialsId: 'aws-root', usernameVariable: "AWS_ACCESS_KEY_ID", passwordVariable: "AWS_SECRET_ACCESS_KEY")]) {
                        sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${REGISTRY_URL}"
                        sh "docker tag ${FRONTEND_IMAGE_REPO}:${FRONTEND_IMAGE_NAME} ${REGISTRY_URL}/${FRONTEND_IMAGE_REPO}:${FRONTEND_IMAGE_NAME}"
                        sh "docker push ${REGISTRY_URL}/${FRONTEND_IMAGE_REPO}:${FRONTEND_IMAGE_NAME}"
                    }
                }
            }
        }
        stage("deploy frontend ") {
            environment {
                AWS_ACCESS_KEY_ID = credentials('jenkins_aws_access_key_id')
                AWS_SECRET_ACCESS_KEY = credentials('jenkins-aws_secret_access_key')
            }
            steps {
                script {
                    echo "Deploying frontend to eks ..."
                    sh "kubectl apply -f kubernetes/frontend.yaml"
                }
            }
        }
    }
    post {
        success {
            echo "PIPELINE SUCCESSFULLY"
        }
        failure {
            echo "PIPELINE FAILED"
        }
    }
}
