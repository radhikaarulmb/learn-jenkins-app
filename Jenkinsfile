pipeline {
    agent any

    environment{
        NETLIFY_SITE_ID = '51da0f9a-656e-4657-ae32-fe4db81196b6'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
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

        stage('Run Tests'){
            parallel{
                stage('Unit Test'){
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        } 
                    }
                    steps{
                        echo 'Test stage'
                        sh'''
                            test -f build/index.html
                            npm test
                        '''
                    }
                    post{
                        always{
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
                    steps{
                        echo 'Test stage'
                        sh'''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test  --reporter=html
                        '''
                    }
                    post{
                        always{
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }

            }
        }

        stage('Deploy Staging') {
                agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        } 
                    }
                    steps {
                        sh'''
                            npm install netlify-cli
                            node_modules/.bin/netlify --version
                            echo "Deploy Staging. Site ID: $NETLIFY_SITE_ID"
                            node_modules/.bin/netlify status
                            node_modules/.bin/netlify deploy --dir=build
                        '''
                    }     
                }

        stage('Deploy Prod') {
                agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        } 
                    }
                    steps {
                        sh'''
                            npm install netlify-cli
                            node_modules/.bin/netlify --version
                            echo "Deploying in Production. Site ID: $NETLIFY_SITE_ID"
                            node_modules/.bin/netlify status
                            node_modules/.bin/netlify deploy --dir=build --prod
                        '''
                    }     
                }


        stage('Prod E2E'){
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        } 
                    }

                    environment{
                        CI_ENVIRONMENT_URL = 'https://silver-sable-fd79d6.netlify.app'
                    } 
                    steps{
                        echo 'Test stage'
                        sh'''
                            npx playwright test  --reporter=html
                        '''
                    }
                    post{
                        always{
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Prod Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
        }    
    }
}
