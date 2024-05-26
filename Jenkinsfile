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
                    ls -la
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
                    echo "Testing Phase"
                    echo `pwd`
                    test -f 'build/index.html'
                    echo "${?}"
                    if [[ "${?}" -eq 0 ]]
                    then
                        echo "Testing the application"
                        npm test
                    fi
                '''
            }
        }
    }
}