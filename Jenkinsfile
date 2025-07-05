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
                        reuseNode true
                    }
                }
            steps {
                sh '''
                    echo "Starting Playwright container"

                    # Stop and remove any existing container with the same name
                    docker rm -f playwright-e2e || true

                    # Run the Playwright container with port 5000 exposed
                    docker run -d --name playwright-e2e \
                        -p 5000:5000 \
                        -v "$PWD":/workspace \
                        -w /workspace \
                        mcr.microsoft.com/playwright:v1.53.0-noble \
                        sh -c "
                            npm ci && \
                            npm install serve && \
                            npm run build && \
                            npx serve -s build -l 5000 --listen 0.0.0.0 & \
                            sleep 10 && \
                            npx playwright install && \
                            npx playwright test --reporter=html
                        "
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