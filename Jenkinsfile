pipeline {
    agent {
        docker {
            image 'node:18-alpine'
            reuseNode true
        }
    }
    stages {
        stage('build') {
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
        stage('Test') {
            steps {
                echo 'Test stage'
                sh '''
                    if [ -f build/index.html ]; then
                        echo "index.html exists in build directory."
                    else
                        echo "index.html does NOT exist in build directory!" >&2
                        exit 1
                    fi
                '''
                sh 'npm test'
            }
        }
    }
}
