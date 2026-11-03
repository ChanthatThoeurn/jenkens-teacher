pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'docker build -t jenkins-react-pipeline .'
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    docker stop reactjs-cont || true
                    docker rm reactjs-cont || true

                    docker run -d \
                      --name reactjs-cont \
                      -p 3005:80 \
                      jenkins-react-pipeline
                '''
            }
        }
    }
}
