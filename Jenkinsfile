pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '47be8439-db59-4a0b-8d3e-d6c8b6357910'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token-yuyongxue') 
        REACT_APP_VERSION = '1.2.3'
    }

    stages {
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

        stage('Tests'){
            parallel {
                stage('Unit Tests'){ 
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps { 
                        sh '''
                            test -f build/index.html
                            npm test
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml' 
                        }
                    }
                }

                stage('E2E'){ 
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true 
                        }
                    }
                    steps { 
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build & 
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always { 
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }  
 
        stage('Deploy Staging'){ 
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true 
                }
            } 

            environment { 
                CI_ENVIRONMENT_URL = 'STAGING_URL_TO_BE_SET'
            }

            steps { 
                sh ''' 
                    npm install netlify-cli node-jq 
                    node_modules/.bin/netlify --version
                    echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json 
                    CI_ENVIRONMENT_URL=$(node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json) 
                    npx playwright test --reporter=html
                '''
            }

            post {
                always { 
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        } 

        stage('Deploy Prod'){ 
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true 
                }
            }

            environment { 
                CI_ENVIRONMENT_URL = 'https://ephemeral-cuchufli-a900c1.netlify.app'
            }

            steps { 
                sh ''' 
                    node --version
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                    npx playwright test --reporter=html
                '''
            }

            post {
                always { 
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Prod E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }  
}
