pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '2761abfd-9b19-4158-88a0-da37ea5ab893'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
    }

    stages {

        stage('Docker') {
            steps {
                sh 'docker build -t my-playwright .'
            }
        }

        stage('build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                echo 'build stage'

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
                            reportName: 'Playwright Local HTML Report',
                            reportTitles: '',
                            useWrapperFileDirectly: true
                            ])
                        }
                    }
                }

            }
        }

        stage('Staging Deploy/E2E') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = "PLACE_HOLDER"
            }

            steps {
                echo 'Staging E2E stage'
                sh '''
                    netlify --version
                    netlify status

                    netlify deploy --dir=build --json > stage-deploy-output.json
                    CI_ENVIRONMENT_URL=$(node-jq -r '.deploy_url' stage-deploy-output.json)

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
                    reportName: 'Playwright Staging HTML Report',
                    reportTitles: '',
                    useWrapperFileDirectly: true
                    ])
                }
            }
        }

        stage('Prod Deploy/E2E') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'https://ma-ata-omer.netlify.app'
            }

            steps {
                sh '''
                    netlify --version
                    netlify status

                    netlify deploy \
                        --prod \
                        --dir=build

                    sleep 5
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
                    reportName: 'Playwright Prod HTML Report',
                    reportTitles: '',
                    useWrapperFileDirectly: true
                    ])
                }
            }
        }

    }

}
