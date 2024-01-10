pipeline {
    environment { // Declaration des variables d'environnements
        DOCKER_ID = "errlog"
        DOCKER_IMAGE_CAST_SERVICE = "jenkins-cast-service"
        DOCKER_IMAGE_MOVIE_SERVICE = "jenkins-movie-service"
        DOCKER_TAG = "v.${BUILD_ID}.0"
    }

    agent any
    stages {
        stage('Docker Build'){
            steps {
                script {
                    sh '''
                        docker rm -f jenkins
                        docker build -t $DOCKER_ID/$DOCKER_IMAGE_CAST_SERVICE:$DOCKER_TAG ./cast-service
                        docker build -t $DOCKER_ID/$DOCKER_IMAGE_MOVIE_SERVICE:$DOCKER_TAG ./movie-service
                        sleep 6
                    '''
                }
            }
        }

        stage('Docker Run built images'){
            steps {
                script {
                    sh '''
                         docker run -d -p 8001:8000 --name cast-service "$DOCKER_ID/$DOCKER_IMAGE_CAST_SERVICE:$DOCKER_TAG" sh -c "uvicorn main:app --reload --host 0.0.0.0 --port 8000"
                         docker run -d -p 8001:8000 --name movie-service "$DOCKER_ID/$DOCKER_IMAGE_MOVIE_SERVICE:$DOCKER_TAG" sh -c "uvicorn main:app --reload --host 0.0.0.0 --port 8000"
                    '''
                }
            }
        }
    }
}
