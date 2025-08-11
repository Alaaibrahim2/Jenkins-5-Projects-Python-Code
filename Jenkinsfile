pipeline {
    agent { label 'worker' }
    environment {
        dockerhub = credentials('docker-cred')
        AWS_REGION = 'us-east-1'
        EKS_CLUSTER = 'my-eks-cluster'
    }
    stages {
        stage('Checkout Code') {
            steps {
                checkout([$class: 'GitSCM',
                          branches: [[name: '*/main']],
                          userRemoteConfigs: [[url: 'https://github.com/Alaaibrahim2/Jenkins-5-Projects-Python-Code.git']]])
            }
        }
        stage('Setup Environment') {
            steps {
                sh '''
                  sudo apt update
                  sudo apt install -y python3-pip python3-venv
                  # بدون تثبيت awscli لأن موجود مسبقاً
                '''
            }
        }
        stage('Docker Version & Build') {
            steps {
                sh 'docker -v'
                sh 'docker build -t alaai/ana:latest .'
            }
        }
        stage('Docker Login & Push') {
            steps {
                sh """
                  echo $dockerhub_PSW | docker login -u $dockerhub_USR --password-stdin
                  docker push alaai/ana:latest
                """
            }
        }
        stage('Describe AWS Instances') {
            steps {
                withAWS(credentials: 'aws-cred', region: env.AWS_REGION) {
                    sh 'aws ec2 describe-instances --region $AWS_REGION'
                }
            }
        }
        stage('Deploy to EKS') {
            steps {
                withAWS(credentials: 'aws-cred', region: env.AWS_REGION) {
                    sh '''
                      aws eks update-kubeconfig --name $EKS_CLUSTER
                      kubectl apply -f Application.yaml
                      
                    '''
                }
            }
        }
    }
}
