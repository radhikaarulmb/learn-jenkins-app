pipeline {
    agent any

    environment{
        NETLIFY_SITE_ID = '51da0f9a-656e-4657-ae32-fe4db81196b6'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
    } 

    stages {

        stage('Docker'){
            steps{
                sh'docker build -t my-playwright .'
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
                            image 'my-playwright'
                            reuseNode true
                        } 
                    }
                    steps{
                        sh'''
                            serve -s build &
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

        stage('Deploy Staging'){
            agent {
                docker  {
                            image 'my-playwright'
                            reuseNode true
                        } 
                    }
            environment{
               CI_ENVIRONMENT_URL = 'STAGING_URL_TO_BE_SET'
            } 
            steps{
                    sh'''
                        netlify --version
                        echo "Deploy Staging. Site ID: $NETLIFY_SITE_ID"
                        netlify status
                        netlify deploy --dir=build --json > deploy-output.json
                        CI_ENVIRONMENT_URL=$(node-jq -r '.deploy_url' deploy-output.json)
                        echo 'CI_ENVIRONMENT_URL'
                        npx playwright test  --reporter=html
                    '''
            }
                post{
                    always{
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
        }

        stage('Deploy Prod'){
                    agent {
                        docker {
                            image 'my-playwright'
                            reuseNode true
                        } 
                    }

                    environment{
                        CI_ENVIRONMENT_URL = 'https://silver-sable-fd79d6.netlify.app'
                    } 
                    steps{
                        sh'''
                            node --version
                            netlify --version
                            echo "Deploying in Production. Site ID: $NETLIFY_SITE_ID"
                            netlify status
                            netlify deploy --dir=build --prod                        
                            npx playwright test  --reporter=html
                        '''
                    }
                    post{
                        always{
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Prod E2E', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }    
            }
        }
