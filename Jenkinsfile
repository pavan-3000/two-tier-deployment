pipeline {
    agent any

    stages {
        stage('git clone') {
            steps {
                git branch: 'main',
                url: 'https://github.com/pavan-3000/two-tier-deployment'
            }
        }

        stage('docker image') {
            steps {
                sh '''
                docker build -t two-tier .
                '''
            }
        }

        stage('make ecr repo') {
            steps {
                withCredentials([string(credentialsId: 'access_key', variable: 'access_key'), string(credentialsId: 'secret_key', variable: 'security_key')]) {
                    sh '''
                        aws configure set aws_access_key_id $access_key
                        aws configure set aws_secret_access_key $security_key
                        aws ecr describe-repositories --repository-name two-tier --region ap-south-1 || \
                        aws ecr create-repository --repository-name two-tier --region ap-south-1 
                    '''
                    
                }
            }
        }

        stage('tag image in ecr') {
            steps {
                withCredentials([string(credentialsId: 'access_key', variable: 'access_key'), string(credentialsId: 'secret_key', variable: 'security_key')]) {
                    sh '''
                    aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 339713101508.dkr.ecr.ap-south-1.amazonaws.com
                    docker tag two-tier:latest 339713101508.dkr.ecr.ap-south-1.amazonaws.com/two-tier:${BUILD_NUMBER}
                    docker tag two-tier:latest 339713101508.dkr.ecr.ap-south-1.amazonaws.com/two-tier:latest
                  '''
                }
            }
        }

        stage('push ecr') {
            steps {
                withCredentials([string(credentialsId: 'access_key', variable: 'access_key'), string(credentialsId: 'secret_key', variable: 'security_key')]) {
                    sh '''
                    docker push 339713101508.dkr.ecr.ap-south-1.amazonaws.com/two-tier:${BUILD_NUMBER}
                    docker push 339713101508.dkr.ecr.ap-south-1.amazonaws.com/two-tier:latest
                    '''
                }
            }
        }

        stage('update k8s image') {
            steps {
                sh '''
                
                sed -i 's|image: *.*|image: 339713101508.dkr.ecr.ap-south-1.amazonaws.com/two-tier:${BUILD_NUMBER}|' ./app-chart/values.yaml
                '''
            }
        }

        stage('Push to GitHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github', passwordVariable: 'pass', usernameVariable: 'user')]) {
                    sh '''
                    git config user.name "pavan-3000"
                    git config user.email "pavansingh3000@gmail.com"

                    # Set the remote URL with credentials
                    git remote set-url origin https://${user}:${pass}@github.com/pavan-3000/two-tier-deployment.git

                    # Add, commit, and push changes
                    git add .
                    git commit -m "Automated commit from Jenkins pipeline"
                    git push origin main
                    '''
                }
            }
        }


    }

}
