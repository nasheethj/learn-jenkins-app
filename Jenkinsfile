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
                cd buiid
                cat index.html
                run npm test
                '''
            }
        }
    }
}
