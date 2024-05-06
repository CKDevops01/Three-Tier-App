pipeline {
    agent any
    tools{
        jdk 'jdk'
        nodejs 'nodejs'
    }
    environment{
        SCANNER_HOME= tool 'sonar'
        AWS_ECR_REPO1= credentials('repo-1')
        AWS_ECR_REPO2= credentials('repo-2')
        AWS_DEFAULT_REGION= 'us-east-1'
        AWS_ACCOUNT_ID= credentials('AWS_Account_ID')
        REPO_URI= 'public.ecr.aws/y2h5i6h0'
    }
    stages {
        stage('Cleaning the Workspace') {
            steps {
                cleanWs()
                }
            }
        }
        stage('Git Checkout') {
            steps {
                script{
                    git branch: 'main', url: '
https://github.com/CKDevops01/Three-Tier-App.git'
                }
            }
        }
        stage('Sonarqube Analysis') {
            steps {
                script{
                    dir('frontend'){
                        withSonarQubeEnv('sonar-server') {
                        sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=three-tier-app \
                        -Dsonar.projectKey=three-tier-app '''
                        }
                    }
                }
            }
        }
        stage('Quality Check') {
            steps {
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar_cred'
                }
            }
        }
        stage('Trivy File Scan') {
            steps {
                dir('frontend'){
                sh 'trivy fs . --format table -o filescan.html'
                sh 'cat filescan.html'
                }
            }
        }
        stage('Docker image build') {
            steps {
                dir('frontend'){
                sh 'docker build -t ${AWS_ECR_REPO1} -f DockerfileM.'
                }
            }
        }
        stage('Docker image buil-2') {
            steps {
                dir('backend'){
                sh 'docker build -t ${AWS_ECR_REPO2} -f DockerfileM.'
                }
            }
        }
        stage('ECR image push') {
            steps {
                dir('frontend'){
                sh 'aws ecr-public get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin public.ecr.aws/y2h5i6h0'
                sh 'docker tag ${AWS_ECR_REPO1}:latest public.ecr.aws/y2h5i6h0/${AWS_ECR_REPO1}:${BUILD_NUMBER}'
                sh 'docker push public.ecr.aws/y2h5i6h0/${AWS_ECR_REPO1}:${BUILD_NUMBER}'
                }
            }
        }

        stage('ECR image push2') {
            steps {
                dir('backend'){
                sh 'aws ecr-public get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${REPO_URI}'
                sh 'docker tag ${AWS_ECR_REPO2}:latest ${REPO_URI}/${AWS_ECR_REPO2}:${BUILD_NUMBER}'
                sh 'docker push ${REPO_URI}/${AWS_ECR_REPO2}:${BUILD_NUMBER}'
                }
            }
        }
         stage('Trivy Image Scan') {
            steps {
                sh 'trivy image ${REPO_URI}/${AWS_ECR_REPO1}:${BUILD_NUMBER}  --format table -o imagescan.html'
                sh 'trivy image ${REPO_URI}/${AWS_ECR_REPO2}:${BUILD_NUMBER}  --format table -o imagescan1.html'
                sh 'cat imagescan.html'
                sh 'cat imagescan1.html'
                }
            }


}
