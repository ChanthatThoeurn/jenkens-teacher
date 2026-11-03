
// pipeline {
//     agent any

//     stages {
//         stage('Checkout') {
//             steps {
//                 checkout scm
//             }
//         }

//         stage('Build') {
//             steps {
//                 sh 'docker build -t jenkins-react-pipeline .'
//             }
//         }

//         stage('Deploy') {
//             steps {
//                 sh '''
//                     set -eu

//                     echo "Step 1: Test the new image..."

//                     docker rm -f reactjs-test 2>/dev/null || true

//                     docker run -d \
//                       --name reactjs-test \
//                       -p 127.0.0.1:3006:3000 \
//                       jenkins-react-pipeline

//                     sleep 5

//                     if ! curl -fsS http://127.0.0.1:3006/ > /dev/null; then
//                         echo "New image failed health check!"
//                         docker logs reactjs-test
//                         docker rm -f reactjs-test
//                         exit 1
//                     fi

//                     echo "New image passed health check."

//                     docker rm -f reactjs-test

//                     echo "Step 2: Preserve the old container..."

//                     docker rm -f reactjs-old 2>/dev/null || true

//                     if docker container inspect reactjs-cont >/dev/null 2>&1; then
//                         docker stop reactjs-cont
//                         docker rename reactjs-cont reactjs-old
//                     fi

//                     echo "Step 3: Start the new application..."

//                     if ! docker run -d \
//                       --name reactjs-cont \
//                       --restart unless-stopped \
//                       -p 3005:3000 \
//                       jenkins-react-pipeline; then

//                         echo "New container failed. Rolling back..."

//                         docker rm -f reactjs-cont 2>/dev/null || true
//                         docker rename reactjs-old reactjs-cont
//                         docker start reactjs-cont
//                         exit 1
//                     fi

//                     sleep 5

//                     if ! curl -fsS http://127.0.0.1:3005/ > /dev/null; then
//                         echo "New application failed. Rolling back..."

//                         docker logs reactjs-cont
//                         docker rm -f reactjs-cont

//                         docker rename reactjs-old reactjs-cont
//                         docker start reactjs-cont

//                         exit 1
//                     fi

//                     echo "Deployment successful!"

//                     docker rm -f reactjs-old 2>/dev/null || true
//                 '''
//             }
//         }
//     }
// }

pipeline {
    agent any

    environment {
    DOCKERHUB_REPO = 'chanthat/reactjs-devop8-template'
    IMAGE_TAG = "${BUILD_NUMBER}"
}

    stages {

        stage('1. Build Docker Image') {
            steps {
                echo 'Building React Docker image...'

                sh '''
                    docker build \
                      -t $DOCKERHUB_REPO:$IMAGE_TAG \
                      -t $DOCKERHUB_REPO:latest .
                '''
            }
        }

        stage('2. Push to Docker Hub') {
            steps {
                echo 'Pushing image to Docker Hub...'

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_TOKEN'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_TOKEN" | docker login \
                          -u "$DOCKER_USER" \
                          --password-stdin

                        docker push $DOCKERHUB_REPO:$IMAGE_TAG
                        docker push $DOCKERHUB_REPO:latest

                        docker logout
                    '''
                }
            }
        }

        stage('3. Deploy to EC2') {
            steps {
                echo 'Deploying React application...'

                sh '''
                    docker pull $DOCKERHUB_REPO:$IMAGE_TAG

                    docker rm -f reactjs-cont 2>/dev/null || true

                    docker run -d \
                      --name reactjs-cont \
                      --restart unless-stopped \
                      -p 3005:3000 \
                      $DOCKERHUB_REPO:$IMAGE_TAG

                    sleep 5

                    curl -f http://127.0.0.1:3005/ \
                      || (docker logs reactjs-cont && exit 1)
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Check Console Output!'
        }
    }
}

