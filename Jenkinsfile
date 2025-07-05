pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'a2721e5e-8111-446d-99ca-569bf8c0c6f3'
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
                sh '''
                ls -ltr
                node --version
                npm --version
                npm ci
                npm run build
                '''
            }
        }
        stage ('Tests') {
            parallel {
                stage('Test') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                        echo "Test Build"
                        pwd
                        ls -ltra
                        cd build
                        cat index.html
                        npm test
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                            }
                        }
                    }
                stage('LOCAL E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.53.0-noble'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                        echo "E2E Test"
                        npm install serve
                        node_modules/.bin/serve -s build &
                        sleep 10
                        npx playwright install
                        npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }
        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                npm install netlify-cli@20.1.1
                node_modules/.bin/netlify --version
                node_modules/.bin/netlify status
                node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }
        stage('PROD E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.53.0-noble'
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL = 'https://scintillating-sunshine-d52894.netlify.app'
            }
            steps {
                sh '''
                echo "PROD Test"
                npx playwright install
                npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Prod Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }
}