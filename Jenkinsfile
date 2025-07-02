pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            pre {
                success{
                    cleanWs()
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
        stage('Test') {
            steps {
                sh '''
                echo "Test Build"
                pwd
                ls -ltra
                cd buiid
                cat index.html
                run npm test
                '''
            }
        }
    }
}
