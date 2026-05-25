pipeline {
    agent any

    environment {
        IMAGE_NAME = "jenkins-app-created"
        CONTAINER_NAME = "jenkins-app-container1"
        SCANNER_HOME = tool 'SonarScanner'
    }

    triggers {
        githubPush()
    }

    stages {
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME .'
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                docker run -d \
                  --name $CONTAINER_NAME \
                  -p 3051:3000 \
                  $IMAGE_NAME
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
                    CI=true npm test -- --watchAll=false
                '''
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                    $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectKey=my-jenkins-app \
                    -Dsonar.sources=. \
                    -Dsonar.host.url=http://192.168.12.101:9000 \
                    -Dsonar.token=YOUR_TOKEN
                    '''
                }
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
