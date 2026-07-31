pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '8327aa66-b9f1-489f-ab8f-f7d0e9f04b5c'
    }

    stages {
        stage('Build') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -lah
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -lah
                '''
            }
        }

        stage('Stage Tests'){
            parallel{
                stage('Unit Test') {
                    agent{
                        docker{
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            ls -lah
                            test -f build/index.html
                            npm run test
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }
                stage('E2E') {
                    agent{
                        docker{
                            image 'mcr.microsoft.com/playwright:v1.62.0-noble'
                            reuseNode true
                            // args '-u root:root' // Don't do this. Bad Security.
                        }
                    }
                    steps {
                        sh '''
                            npm install serve
                            npx playwright install chromium
                            node_modules/.bin/serve -s build &
                            sleep .25
                            npx playwright test --reporter=html
                            #  Need to make this work.... System.setProperty("hudson.model.DirectoryBrowserSupport.CSP", "sandbox allow-scripts;")

                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }
        
        stage('Deploy') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli@20.1.1 
                    node-modules/.bin/netlify --version
                    echo "Deploying to Netlify Site ID: ${NETLIFY_SITE_ID}"
                '''
            }
        }
    }
}
