pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                } 
            }
            steps {
                sh'''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
        stage('Test'){
            steps{
                echo 'Test stage'
                sh'''
                    find /workspaces/learn-jenkins-app/build -name index.html
                    npm test
                '''
            }
        }
    }
}
