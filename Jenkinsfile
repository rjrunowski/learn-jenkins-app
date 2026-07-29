pipeline {
    agent any

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
        stage('Test') {
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
        }
        stage('E2E') {
            agent{
                docker{
                    image 'mcr.microsoft.com/playwright:v1.62.0-noble'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install -g serve
                    serve -s build
                    npx playwright test
                '''
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'build/**', allowEmptyArchive: true
            junit 'test-results/junit.xml'
        }
    }
}
