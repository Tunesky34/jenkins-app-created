// jenkins-docker pipeline script

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
                    npm ci --cache /tmp/.npm-cache
                    npm run build
                    ls -la
                '''
            }
        }

        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    test -f build/index.html
                    CI=true npm test
                '''
            }
        }

        stage('Deploy to Render') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                withCredentials([string(credentialsId: 'RENDER_API_KEY', variable: 'RENDER_API_KEY')]) {
                    sh '''
                        apk add --no-cache curl
                        SERVICE_ID=$(echo $RENDER_API_KEY | cut -d':' -f1)
                        curl -X POST https://api.render.com/v1/services/${SERVICE_ID}/deploys \
                          -H "Authorization: Bearer $RENDER_API_KEY" \
                          -H "Content-Type: application/json" \
                          -d '{"clearCache":"clear"}'
                    '''
                }
            }
        }
        
    }

    post {
        always {
            junit 'test-results/junit.xml'
        }
        success {
            echo 'Pipeline completed - app deployed to Render!'
        }
        failure {
            echo 'Pipeline failed - deployment skipped.'
        }
    }
}