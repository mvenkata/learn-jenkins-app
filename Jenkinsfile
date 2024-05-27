pipeline {
    agent any

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
                            echo "E2E Phase"
                            npm install serve
                            ## Run in background in order to avoid stuck in jenkins
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }

                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwrite HTML Report', reportTitles: '', useWrapperFileDirectly: true])
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
                   ## do not use -g arg which will wail with access.
                   npm install netlify-cli
                   test -f ./node_modules/.bin/netlify
                   ./node_modules/.bin/netlify/netlify --version
                '''
            }
        }
    }
}