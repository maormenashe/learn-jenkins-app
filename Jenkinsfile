pipeline {
    agent any

    environment {
        NETLIFY_CI_ID = '2761abfd-9b19-4158-88a0-da37ea5ab893'
    }

    stages {
        stage('build') {
            agent {
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
        
        stage('Tests') {
            parallel() {

                stage('Unit Tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
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

                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }

                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    steps {
                        echo 'E2E stage'
                        sh '''
                            npm install serve
                            npx serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }

                    post {
                        always {
                            publishHTML([
                            allowMissing: false,
                            alwaysLinkToLastBuild: false,
                            icon: '',
                            keepAll: false,
                            reportDir: 'playwright-report',
                            reportFiles: 'index.html',
                            reportName: 'Playwright HTML Report',
                            reportTitles: '',
                            useWrapperFileDirectly: true
                            ])
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
                    #npm install netlify-cli --save-dev
                    npm install netlify-cli@20.1.1

                    npx netlify --version

                    echo "Deploying to production site ID: ${env.NETLIFY_CI_ID}"
                    #npx netlify deploy --prod --dir=build
                '''
            }
        }

    }

}
