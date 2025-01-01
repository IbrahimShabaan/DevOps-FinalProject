pipeline {
    agent any

    environment {
        
        APP_CREDENTIALS = credentials('mycredential')  
        FRONTEND_IMAGE = "ibrahimshaaban/final-project:frontend.${BUILD_NUMBER}" 
        BACKEND_IMAGE = "ibrahimshaaban/final-project:backend.${BUILD_NUMBER}"   
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo 'Cloning the repository...'
                git branch: 'main', url: 'https://github.com/ibrahimshaaban/DevOpsSphere-finalProject' 
            }
        }

        stage('Build Docker Images') {
            parallel {
                stage('Build Backend Docker Image') {
                    steps {
                        echo "Building Backend Docker Image: ${BACKEND_IMAGE}"
                        dir('backend-flask') {  
                            sh '''
                                docker build -t $BACKEND_IMAGE .
                                echo $APP_CREDENTIALS_PSW | docker login -u $APP_CREDENTIALS_USR --password-stdin
                                docker push $BACKEND_IMAGE
                            '''
                        }
                    }
                }
                stage('Build Frontend Docker Image') {
                    steps {
                        echo "Building Frontend Docker Image: ${FRONTEND_IMAGE}"
                        dir('frontend-html') {  
                            sh '''
                                docker build -t $FRONTEND_IMAGE .
                                echo $APP_CREDENTIALS_PSW | docker login -u $APP_CREDENTIALS_USR --password-stdin
                                docker push $FRONTEND_IMAGE
                            '''
                        }
                    }
                }
            }
        }

        stage('Apply Kubernetes Manifests') {
            steps {
                echo 'Applying Kubernetes manifests with updated image tags...'
                sh '''
                    sed -i "s|image: .*backend.*|image: $BACKEND_IMAGE|g" backend-deployment.yml
                    sed -i "s|image: .*frontend.*|image: $FRONTEND_IMAGE|g" frontend-deployment.yml

                    kubectl apply -f K8s-manifest/backend-configmap.yml
                    kubectl apply -f K8s-manifest/backend-pvc.yml
                    kubectl apply -f K8s-manifest/backend-deployment.yml
                    kubectl apply -f K8s-manifest/backend-service.yml
                    kubectl apply -f K8s-manifest/sql-configmap.yml
                    kubectl apply -f SealedSecrets/sql-sealedSecret.yml
                    kubectl apply -f K8s-manifest/my-pv.yml
                    kubectl apply -f K8s-manifest/mysql-pvc.yml
                    kubectl apply -f K8s-manifest/mysql-deployment.yml
                    kubectl apply -f K8s-manifest/mysql-service.yml
                    kubectl apply -f SealedSecrets/pullingSealedSecret.yml
                    kubectl apply -f K8s-manifest/frontend-configmap.yml
                    kubectl apply -f K8s-manifest/frontend-deployment.yml
                    kubectl apply -f K8s-manifest/frontend-service.yml
                    kubectl apply -f K8s-manifest/ingress-resource.yml
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Please check the logs for details.'
        }
    }
}
