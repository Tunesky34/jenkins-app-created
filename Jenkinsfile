pipeline {
    agent any

    environment {
        IMAGE_NAME = "jenkins-app-created"
        CONTAINER_NAME = "jenkins-app-container1"
    }

    tools {
        nodejs 'NodeJS'
    }

    triggers {
        githubPush()
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Run Tests') {
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
            environment {
                SCANNER_HOME = tool 'SonarScanner'
            }

            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectKey=my-jenkins-app \
                        -Dsonar.sources=. \
                        -Dsonar.sourceEncoding=UTF-8
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t $IMAGE_NAME .
                '''
            }
        }

        stage('Stop Existing Container') {
            steps {
                sh '''
                    docker stop $CONTAINER_NAME || true
                    docker rm $CONTAINER_NAME || true
                '''
            }
        }

        stage('Run Docker Container') {
            steps {
                sh '''
                    docker run -d \
                    --name $CONTAINER_NAME \
                    -p 3051:3000 \
                    $IMAGE_NAME
                '''
            }
        }

        stage('Deploy to Render & Netlify') {
            steps {
                withCredentials([
                    string(credentialsId: 'RENDER_HOOK', variable: 'RENDER_HOOK'),
                    string(credentialsId: 'NETLIFY_HOOK', variable: 'NETLIFY_HOOK')
                ]) {

                    sh '''
                        curl "$RENDER_HOOK"

                        curl -X POST -d {} "$NETLIFY_HOOK"
                    '''
                }
            }
        }
    }

    post {

        always {
            echo 'Pipeline execution finished.'
        }

        success {
            echo 'Pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed.'
        }
    }
}