pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
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
    }
}
