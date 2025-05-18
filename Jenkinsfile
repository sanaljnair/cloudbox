pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git credentialsId: 'deploy-html', url: 'https://github.com/sanaljnair/cloudbox.git'
            }
        }
        stage('Build and Push Docker Image') {
            steps {
                script {
                    // Set ECR repository details
                    def awsRegion = 'us-east-1'   
                    def ecrRepo = '594154735784.dkr.ecr.us-east-1.amazonaws.com/upgrad-repo'  
                    def imageName = 'my-html-app' // The name you want to give to the image
                    def imageTag = "${env.BUILD_NUMBER}" // Tag the image with the Jenkins build number

                    // Authenticate with ECR
                    withAWS(region: awsRegion, credentials: '') { // Use empty credentials, relies on IAM role
                        sh "aws ecr get-login-password --region ${awsRegion} | docker login --username AWS --password-stdin ${ecrRepo.substring(0, ecrRepo.indexOf('/'))}"
                    }

                    // Build the Docker image
                    sh "docker build -t ${imageName} ."

                    // Tag the image for ECR
                    sh "docker tag ${imageName}:latest ${ecrRepo}:${imageTag}"

                    // Push the image to ECR
                    sh "docker push ${ecrRepo}:${imageTag}"

                    // Set the IMAGE_TAG environment variable for use in subsequent stages
                    env.IMAGE_TAG = imageTag
                    env.ECR_REPO = ecrRepo
                }
            }
        }
        stage('Deploy to EC2') {
            steps {
                sshagent (credentials: ['ec2-ssh-credentials']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ubuntu@<your_ec2_instance_ip> <<EOF
                            echo "Pulling and deploying Docker image from ECR..."
                            # Use docker-compose
                            docker-compose -f docker-compose.yml pull
                            docker-compose -f docker-compose.yml up -d --force-recreate
                            echo "Deployment complete."
                        EOF
                    """
                }
            }
        }
    }
}