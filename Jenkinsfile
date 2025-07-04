pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'a2721e5e-8111-446d-99ca-569bf8c0c6f3'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
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
                        npx playwright install chromium
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
        stage('Deploy Staging') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                npm install netlify-cli@20.1.1 node-jq
                node_modules/.bin/netlify --version
                node_modules/.bin/netlify status
                node_modules/.bin/netlify deploy --dir=build --json > json_output.txt
                node_modules/.bin/node-jq -r '.deploy_url' json_output.txt > deploy_id.txt
                '''
                script {
                    def deployUrl = readFile('deploy_id.txt').trim()
                    echo "Staging Deployment URL: ${deployUrl}"
                    env.STAGING_URL = sh(script: 'cat deploy_id.txt', returnStdout: true).trim()
                }
            }
        }
        stage('Staging E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.53.0-noble'
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL = "${env.STAGING_URL}"
            }
            steps {
                sh '''
                echo "PROD Testing"
                npx playwright install chromium
                npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Staging Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
        stage('Approve') {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    input cancel: 'Not Yet', message: 'Approve?', ok: 'Ready to Proceed'
                }
            }
        }
        stage('Deploy PROD') {
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
                echo "PROD Testing"
                npx playwright install chromium
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
