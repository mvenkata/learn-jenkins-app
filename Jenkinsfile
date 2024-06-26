pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '5254a181-7512-4aab-82c8-5d7bb29001ee'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.2.${BUILD_ID}"
    }

    stages {
        // stage('Docker') {
        //     steps {
        //         sh 'docker build -t my-playwright .'
        //     }
        // }
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
                            //image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            image 'my-playwright'
                            reuseNode true
                            // Donot run it as root and it is not an good idea
                            //args '-u root:root'
                        }
                    }
                    steps {
                        sh '''
                            echo "E2E Phase..."
                            ##npm install serve
                            ## Run in background in order to avoid stuck in jenkins
                            serve -s build &
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
            /*
                Mergeted both deploy and E2E staging
                This is End2End staging phase with docker
            */
            agent {
                docker {
                    //image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    image 'my-playwright'
                    reuseNode true
                    // Donot run it as root and it is not an good idea
                    //args '-u root:root'
                }
            }

            environment {
                // We need this, since property file sets localhost
                CI_ENVIRONMENT_URL='STAGEING_TO_BE_SET'
            }
            
            steps {
                sh '''
                    echo 'Deploy Staging Phase'
                    node --version
                    ## do not use -g arg which will wail with access.
                    ##npm install netlify-cli node-jq
                    ##test -f ./node_modules/.bin/netlify
                    netlify --version
                    echo "Deploying to Staging, Site ID: ${NETLIFY_SITE_ID}"
                    ##./node_modules/.bin/netlify status
                    netlify status
                    ## removed the prod flag and all others are same as prod
                    ##./node_modules/.bin/netlify deploy --dir=build --json | tee deploy-output.json
                    netlify deploy --dir=build --json | tee deploy-output.json
                    ##CI_ENVIRONMENT_URL=$(./node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json)
                    CI_ENVIRONMENT_URL=$(node-jq -r '.deploy_url' deploy-output.json)
                    echo "Prod E2E Phase..."
                    npx playwright test --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwrite E2E Staging Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }    

        // stage('Approval') {
        //     steps {
        //         sh '''
        //             echo "Approval stage"
        //         '''
        //         timeout(time: 15, unit: 'MINUTES') {
        //             input message: 'Do you wish to deploy to production?', ok: 'Yes, I am sure!'
        //         }
        //     }

        // }

        stage('Deploy Prod') {          
            /*
                This is End2End test phase with docke
            */
            agent {
                docker {
                    // This image has node, hence merging
                    //image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    image 'my-playwright'
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
                    node --version
                    echo "Prod Deploy Phase"
                    ## do not use -g arg which will wail with access.
                    ##npm install netlify-cli
                    ##test -f ./node_modules/.bin/netlify
                    netlify --version
                    echo "Deploying to production, Site ID: ${NETLIFY_SITE_ID}"
                    netlify status
                    netlify deploy --dir=build --prod
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