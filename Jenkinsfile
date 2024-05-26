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
        }

        stage('E2E') {
            /*
                This is End2End test phase with docke
            */
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.44.1-jammy'
                    reuseNode true
                }
            }
             steps {
                sh '''
                    echo "E2E Phase"
                    npm install -g serve
                    serve -s build
                    npx playwright test
                '''
            }
        }    
    }

    post {
        always {
            junit 'test-results/junit.xml'
        }
    }
}