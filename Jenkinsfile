pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '5254a181-7512-4aab-82c8-5d7bb29001ee'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
        // This is build phase
        stage('Build') {
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
                '''
            }
        }

        stage('Run Tests') {
            parallel {
                stage('Test') {
                    /*
                        This is test phase with docke
                    */
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            echo "Testing Phase"
                            echo `pwd`
                            test -f 'build/index.html'
                            # echo "${?}"
                            npm test
                        '''
                    }

                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }

                stage('E2E') {
                    /*
                        This is End2End test phase with docke
                    */
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                            // Donot run it as root and it is not an good idea
                            //args '-u root:root'
                        }
                    }
                    steps {
                        sh '''
                            echo "E2E Phase..."
                            npm install serve
                            ## Run in background in order to avoid stuck in jenkins
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }

                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwrite Local Report', reportTitles: '', useWrapperFileDirectly: true])
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
                   ## do not use -g arg which will wail with access.
                   npm install netlify-cli
                   test -f ./node_modules/.bin/netlify
                   ./node_modules/.bin/netlify --version
                   echo "Deploying to Staging, Site ID: ${NETLIFY_SITE_ID}"
                   ./node_modules/.bin/netlify status
                   ## removed the prod flag and all others are same as prod
                   ./node_modules/.bin/netlify deploy --dir=build


                '''
            }
        }

        stage('Approval') {
            steps {
                sh '''
                    echo "Approval stage"
                '''
                timeout(time: 15, unit: 'MINUTES') {
                    input message: 'Do you wish to deploy to production?', ok: 'Yes, I am sure!'
                }
            }

        }
        stage('Deploy Prod') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                   ## do not use -g arg which will wail with access.
                   npm install netlify-cli
                   test -f ./node_modules/.bin/netlify
                   ./node_modules/.bin/netlify --version
                   echo "Deploying to production, Site ID: ${NETLIFY_SITE_ID}"
                   ./node_modules/.bin/netlify status
                   ./node_modules/.bin/netlify deploy --dir=build --prod


                '''
            }
        }

        stage('Prod E2E') {          
            /*
                This is End2End test phase with docke
            */
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                    // Donot run it as root and it is not an good idea
                    //args '-u root:root'
                }
            }

            environment {
                CI_ENVIRONMENT_URL='https://elegant-pika-614d48.netlify.app'
            }
            
            steps {
                sh '''
                    echo "Prod E2E Phase..."
                    npx playwright test --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwrite E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }    
    }
}