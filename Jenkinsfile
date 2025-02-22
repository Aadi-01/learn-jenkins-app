pipeline {
    agent any
    environment {
        NETLIFY_SITE_ID = '5c65ad5c-4ac4-4950-b5e8-aef241157208'
        NETLIFY_AUTH_TOKEN= credentials('netlify-token')
    }
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
        /*
        stage('Lint') {
            agent{
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install eslint
                    npx eslint . --ext .js,.jsx,.ts,.tsx
                '''
            }
        }
        */
        stage('Run Tests Parallel'){
            parallel{
                stage('Unit Test'){
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
                    post {
                        always {
                        //  junit 'reports/**/*.xml'
                            junit 'jest-results/junit.xml'
                            // publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                            // publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: 'Playwright test', useWrapperFileDirectly: true])
                        }
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
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                    always {
                        // junit 'reports/**/*.xml'
                        // junit 'jest-results/junit.xml'
                        // publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: 'Playwright test', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }
        stage('Deploy to staging env') {
            agent{
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                npm install netlify-cli
                node_modules/.bin/netlify --version
                echo "Deploying to Staging. Site ID: $NETLIFY_SITE_ID"
                node_modules/.bin/netlify status
                node_modules/.bin/netlify deploy --dir=build
                '''
            }
        }
        stage('Deploy Prod') {
            agent{
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                npm install netlify-cli
                node_modules/.bin/netlify --version
                echo "Deploying to Netlify- Production. Site ID: $NETLIFY_SITE_ID"
                node_modules/.bin/netlify status
                node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }
        stage('E2E-Production'){
                    agent{
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    environment{
                            CI_ENVIRONMENT_URL= 'https://jovial-tarsier-6ec8f8.netlify.app'
                    }
                    steps {
                        sh'''
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Prod E2E', reportTitles: 'Playwright E2E Prod test', useWrapperFileDirectly: true])
                        }
                    }
        }
    }
    
}
