pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '8327aa66-b9f1-489f-ab8f-f7d0e9f04b5c'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_NUMBER"
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
                            image 'learn-playwright'
                            reuseNode true
                            // args '-u root:root' // Don't do this. Bad Security.
                        }
                    }
                    steps {
                        sh '''
                            npm install serve
                            npx playwright install chromium
                            node_modules/.bin/serve -s build &
                            # sleep .25
                            npx playwright test --reporter=html
                            #  Need to make this work.... System.setProperty("hudson.model.DirectoryBrowserSupport.CSP", "sandbox allow-scripts;")

                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }
        
        stage('Deploy Stage') {
            agent {
                docker {
                    image 'learn-playwright'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    netlify --version
                    echo "Deploying to Stage Site ID: ${NETLIFY_SITE_ID}"
                    netlify status
                    netlify deploy --dir=build --auth=$NETLIFY_AUTH_TOKEN --site=$NETLIFY_SITE_ID --json > deploy-output.json
                    cat deploy-output.json
                '''
                script {
                    env.CI_ENVIRONMENT_URL = sh(script: "node -e \"console.log(require('./deploy-output.json').deploy_url)\"", returnStdout: true).trim()
                }
                echo "Stage URL: ${env.CI_ENVIRONMENT_URL}"
                sh '''
                    npx playwright install chromium
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Stage E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

        stage('Deploy Prod') {
            agent{
                docker{
                    image 'learn-playwright'
                    reuseNode true
                    // args '-u root:root' // Don't do this. Bad Security.
                }
            }
            environment {
                CI_ENVIRONMENT_URL = 'https://runowski.netlify.app'
            }
            steps {
                sh '''
                    npx playwright install chromium
                    echo "Deploying to Netlify Site ID: ${NETLIFY_SITE_ID}"
                    netlify status
                    netlify deploy --prod --dir=build --auth=$NETLIFY_AUTH_TOKEN --site=$NETLIFY_SITE_ID --skip-functions-cache
                    npx playwright test --reporter=html

                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Prod E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }
}