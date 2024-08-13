pipeline {
    agent any

    environment { 
        REACT_APP_VERSION = "1.0.$BUILD_ID"
        AWS_DEFAULT_REGION = 'ap-southeast-1'
    }

    stages {  

         stage('Deploy to AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    reuseNode true
                    args "-u root --entrypoint=''"
                }
            } 

            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws-iam-yuyongxue-dev', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version 
                        yum install jq -y
                        LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq '.taskDefinition.revision')
                        echo "Task Definition Revision: ${LATEST_TD_REVISION}"
                        aws ecs update-service --cluster LearnJenkinsApp-Cluster-Prod --service LearnJenkinsApp-Serivice-Prod --task-definition LearnJenkinsApp-TaskDefinition-Prod:$LATEST_TD_REVISION
                        aws ecs wait services-stable --cluster LearnJenkinsApp-Cluster-Prod --service LearnJenkinsApp-Serivice-Prod
                    '''
                }                
            }
        }  

        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
        
            steps {
                sh '''
                    echo 'Making change to trigger deploy'
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            } 
        }            
    }  
}
