pipeline {
    agent any

    environment {
        AWS_REGION    = "ap-south-1"   // change to your region
        AWS_ACCOUNT   = "<your-account-id>"
        ECR_REPO      = "jg-jwr-config-server"
        IMAGE_TAG     = "${env.BUILD_NUMBER}"
    }

    tools {
        maven 'MAVEN_HOME'   // configure Maven in Jenkins Global Tools
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: "${env.BRANCH_NAME}", 
                    url: 'https://bitbucket.org/<workspace>/jg-jwr-config-server.git',
                    credentialsId: 'bb-credentials'
            }
        }

        stage('Build JAR') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
            post {
                success {
                    archiveArtifacts artifacts: "target/*.jar", fingerprint: true
                }
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    dockerImage = docker.build("${AWS_ACCOUNT}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}")
                }
            }
        }

        stage('Login to ECR & Push') {
            steps {
                script {
                    sh """
                      aws ecr get-login-password --region ${AWS_REGION} \
                        | docker login --username AWS --password-stdin ${AWS_ACCOUNT}.dkr.ecr.${AWS_REGION}.amazonaws.com
                    """
                    dockerImage.push()
                    dockerImage.push("latest")   // also push latest tag
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
