pipeline {
    agent any

    stages {
        agent {
            docker {
                image 'node:18-alpine'
                reuseNode true
            }
        }
        stage('Hello') {
            steps {
                sh '''
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
