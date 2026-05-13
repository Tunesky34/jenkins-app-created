// jenkins-docker pipeline script

pipeline {
    agent any

    environment{
        RENDER_API_KEY: "rnd_dTg7XpJ5oR0nUmhxLDytpnSEaP40"
    }

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
                sh '''
                    curl -fsSL https://render.com/install-cli.sh | bash
                        render --version
                        echo "Deployment to production. Site ID: $RENDER_API_KEY"
                    '''
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