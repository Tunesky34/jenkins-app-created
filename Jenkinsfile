pipeline {
    agent any
    
    environment {
        Image_name = "jenkins-practice-image"
        Container_name = "jenkins-practice-container2"
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
        stage('Deploy to Render') {
            steps {
                withCredentials([string(credentialsId: 'render-credentials', variable: 'RENDER_HOOK')]) {
                    sh 'curl --fail --show-error --request POST "$RENDER_HOOK"'
                }
            }
        }
  //      stage('Deploy to Netlify') {
   //         steps {
   //         withcredentials{
 //               usernamePassword(credentialsId: 'netlify-credentials', usernameVariable: 'NETLIFY_USERNAME', passwordVariable: 'NETLIFY_PASSWORD')
    //        }
    //            script {
     //               sh 'curl -X POST $NETLIFY_HOOK'
                
      //          }
   //            }
   //        }
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
