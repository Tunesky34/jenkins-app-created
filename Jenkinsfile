// jenkins-docker pipeline script

pipeline {
    agent any

    triggers {
        githubPush()
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
                    image 'node:18'
                    reuseNode true
                }
            }
            steps {     
                withCredentials([
                    string(credentialsId: 'RENDER_HOOK', variable: 'RENDER_HOOK'), string(credentialsId: 'NETLIFY_HOOK', variable: 'NETLIFY_HOOK')]) {
                    sh '''
                        curl "$RENDER_HOOK"
                        curl -X POST -d {} https://api.netlify.com/build_hooks/6a09778699bfb3b415ff0bbb
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