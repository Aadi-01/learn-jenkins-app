pipeline {
    agent any

    stages {
        // this is a build stage
        stage('Build') {
            agent{
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
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
        
        stage('Test'){
            agent{
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh'''
                #echo "Test stage"
                test -f build/index.html
                npm test
                '''
            }
        }
        stage('E2E'){
            agent{
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            steps {
                sh'''
                    npm install serve
                    node_modules/.bin/serve -s build &
                    sleep 20
                    npx playwright test
                '''
            }
        }
    }
    post {
        always {
        //    junit 'reports/**/*.xml'
            junit 'jest-results/junit.xml'
        }
    }
}
