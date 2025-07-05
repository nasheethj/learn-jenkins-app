pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'a2721e5e-8111-446d-99ca-569bf8c0c6f3'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
        stage('LOCAL E2E') {
            agent {
                docker {
                    image 'node:18-alpine'
                    image 'mcr.microsoft.com/playwright:v1.53.0-noble'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "Installing dependencies and starting React app"

                    npm ci
                    npm install serve

                    echo "Building React app"
                    npm run build

                    echo "Serving React app on http://localhost:5000"
                    npx serve -s build -l 5000 --listen 0.0.0.0 &
                    sleep 10

                    echo "Waiting for the app to be available..."
                    npx wait-on http://localhost:5000

                    echo "Running Playwright tests"
                    npx playwright install
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        keepAll: false,
                        reportDir: 'playwright-report',
                        reportFiles: 'index.html',
                        reportName: 'Playwright Local Report',
                        reportTitles: '',
                        useWrapperFileDirectly: true
                    ])
                }
            }
        }
    }
}