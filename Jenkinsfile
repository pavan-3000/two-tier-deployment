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

     
        stage('Update Kubernetes Image') {
            steps {
                sh '''
                    # Print the BUILD_NUMBER to ensure it's set correctly
                    echo "BUILD_NUMBER: ${BUILD_NUMBER}"
                    
                    # Print the image line before applying sed
                    echo "Before sed:"
                    cat ./app-chart/values.yaml | grep image:
        
                    # Update the image tag with the build number
                    sed -i "s|image: *.*|image: 339713101508.dkr.ecr.ap-south-1.amazonaws.com/two-tier:${BUILD_NUMBER}|" ./app-chart/values.yaml

        
                    # Print the image line after applying sed
                    echo "After sed:"
                    cat ./app-chart/values.yaml | grep image:
        
                    # Check if there are changes to commit
                    git diff ./app-chart/values.yaml
                    git status
        
                    # Explicitly add the file and commit if there are changes
                    git add ./app-chart/values.yaml
                    git commit -m "Updated image"
                '''
            }
        }
        
        stage('Push to GitHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github', passwordVariable: 'password', usernameVariable: 'name')]) {
                    sh '''
                        git config user.name "pavan-3000"
                        git config user.email "pavansingh3000@gmail.com"
                        # Set the remote URL with credentials
                        echo ${name}
                        echo "Using remote URL: https://${name}:${password}@github.com/pavan-3000/two-tier-deployment.git"
                        git remote set-url origin https://${name}:${password}@github.com/pavan-3000/two-tier-deployment.git    
                        git push origin main
                    '''
                }
            }
        }

       


    }

}
