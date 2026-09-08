pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    
    environment {
        // Defined as strings to prevent Groovy type-casting issues
        AWS_ACCOUNT_ID = '125840291232'
        AWS_REGION     = 'ap-south-2'
        ECR_REPO_NAME  = 'boardgame'
        IMAGE_TAG      = "${BUILD_NUMBER}"
        
        // Construct the registry URL once to avoid typos later
        ECR_REGISTRY   = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
    }
    
    stages {
        stage('Build & Test') {
            steps {
                // 'clean package' handles compiling, testing, and packaging in one command
                sh 'mvn clean package'
            }
        }
        
        stage('Build & Push Docker Image') {
            steps {
                // Use triple double-quotes (""") for Groovy variable interpolation
                sh """
                    # Login to ECR
                    # Note the escape slash on \$TOKEN so bash handles it, not Groovy
                    TOKEN=\$(aws ecr get-login-password --region ${AWS_REGION})
                    docker login --username AWS -p "\$TOKEN" ${ECR_REGISTRY}
                    
                    # Build Docker image
                    docker build -t ${ECR_REPO_NAME}:${IMAGE_TAG} .
                    
                    # Tag for ECR
                    docker tag ${ECR_REPO_NAME}:${IMAGE_TAG} ${ECR_REGISTRY}/${ECR_REPO_NAME}:${IMAGE_TAG}
                    docker tag ${ECR_REPO_NAME}:${IMAGE_TAG} ${ECR_REGISTRY}/${ECR_REPO_NAME}:latest
                    
                    # Push to ECR
                    docker push ${ECR_REGISTRY}/${ECR_REPO_NAME}:${IMAGE_TAG}
                    docker push ${ECR_REGISTRY}/${ECR_REPO_NAME}:latest
                """
            }
        }
    }
    
    post {
        always {
            // Generate test report regardless of pipeline success/failure
            junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml'
            
            // Cleanup local images to prevent Jenkins disk space exhaustion
            sh """
                docker rmi ${ECR_REPO_NAME}:${IMAGE_TAG} || true
                docker rmi ${ECR_REGISTRY}/${ECR_REPO_NAME}:${IMAGE_TAG} || true
                docker rmi ${ECR_REGISTRY}/${ECR_REPO_NAME}:latest || true
            """
        }
    }
}
