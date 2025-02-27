pipeline {
    agent any
    environment {
        NETLIFY_SITE_ID = '5c65ad5c-4ac4-4950-b5e8-aef241157208'
        NETLIFY_AUTH_TOKEN= credentials('netlify-token')
        REACT_APP_VERSION= "1.0.$BUILD_ID"
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
                            image 'my-playwright'
                            reuseNode true
                        }
                    }
                    steps {
                        sh'''
                            serve -s build &
                            sleep 20
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                    always {
                        // junit 'reports/**/*.xml'
                        // junit 'jest-results/junit.xml'
                        // publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Local E2E', reportTitles: 'Playwright test', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }
        stage('Deploy Stage'){
                    agent{
                        docker {
                            image 'my-playwright'
                            reuseNode true
                        }
                    }
                    environment{
                            // CI_ENVIRONMENT_URL= 'https://jovial-tarsier-6ec8f8.netlify.app'
                            CI_ENVIRONMENT_URL= 'STAGE URL TO BE REPLACED'
                    }
                    steps {
                        sh'''
                        netlify --version
                        echo "Deploying to Staging. Site ID: $NETLIFY_SITE_ID"
                        netlify status
                        netlify deploy --dir=build --json > deploy-output.json
                        CI_ENVIRONMENT_URL=$(node-jq -r '.deploy_url' deploy-output.json)
                        sleep 20
                        npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E', reportTitles: 'Staging E2E', useWrapperFileDirectly: true])
                        }
                    }
        }
        stage('Deploy Production') {
                    agent{
                        docker {
                            image 'my-playwright'
                            reuseNode true
                        }
                    }
                    environment{
                            CI_ENVIRONMENT_URL= 'https://jovial-tarsier-6ec8f8.netlify.app'
                    }
                    steps {
                        sh'''
                        node --version
                        netlify --version
                        echo "Deploying to Netlify- Production. Site ID: $NETLIFY_SITE_ID"
                        netlify status
                        netlify deploy --dir=build --prod
                        sleep 20
                        npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Production E2E', reportTitles: 'Prod E2E', useWrapperFileDirectly: true])
                        }
                    }
        }
    }

}
