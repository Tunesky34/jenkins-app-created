pipeline {
    agent any
    
    environment {
        Image_name = "jenkins-practice-image"
        Container_name = "jenkins-practice-container4"
    }

    triggers {
        pollSCM('* * * * *') // Polls the SCM every minute for changes
    }    
    stages {
        stage('Build docker image') {
            steps {
                script {
                    sh 'docker build -t $Image_name .'
                }
            }
        }

        stage('Run container') {
            steps {
                script {
                    sh 'docker run -d --name $Container_name $Image_name'
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    sh 'docker exec $Container_name echo "Running tests..."'
                    // Add your test commands here
                }
            }
        }

        stage('sonarqube analysis') {
            
            steps {
                script {
                    sh 'docker exec $Container_name echo "Running SonarQube analysis..."'
                    // Add your SonarQube analysis commands here
                }
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
    post{
        success{
            echo 'Pipeline completed successfully.'
        }
        failure{
            echo 'Pipeline failed.'
        }
    }



}
