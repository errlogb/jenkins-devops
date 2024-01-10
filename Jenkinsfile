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
            parallel {
                stage('Build cast service') {
                    steps {
                        sh '''
                            docker build -t $DOCKER_ID/$DOCKER_IMAGE_CAST_SERVICE:$DOCKER_TAG ./cast-service
                        '''
                    }
                }
                stage('Build movie service') {
                    steps {
                        sh '''
                            docker build -t $DOCKER_ID/$DOCKER_IMAGE_MOVIE_SERVICE:$DOCKER_TAG ./movie-service
                        '''
                    }
                }
            }
        }

        stage('Docker Test Run'){
            parallel {
                stage('Run movie service') {
                    steps {
                        sh '''
                            docker network create movie_network
                            docker run --name movie_db -e POSTGRES_USER=movie_db_username -e POSTGRES_PASSWORD=movie_db_password -e POSTGRES_DB=movie_db_dev -d postgres:12.1-alpine
                            docker run --name movie_service -d -p 8001:8000 -e DATABASE_URI=postgresql://movie_db_username:movie_db_password@movie_db/movie_db_dev "$DOCKER_ID/$DOCKER_IMAGE_MOVIE_SERVICE:$DOCKER_TAG" sh -c "uvicorn app.main:app --reload --host 0.0.0.0 --port 8000"
                            docker network connect movie_network movie_db
                            docker network connect movie_network movie_service
                            docker exec -d movie_service uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
                            sleep 6
                            curl http://localhost:8001/api/v1/movies/docs
                            docker container rm movie_service -f
                            docker container rm movie_db -f
                            docker network rm movie_network -f
                        '''
                    }
                }
                stage('Run cast service') {
                    steps {
                        sh '''
                            docker network create cast_network
                            docker run --name cast_db -e POSTGRES_USER=cast_db_username -e POSTGRES_PASSWORD=cast_db_password -e POSTGRES_DB=cast_db_dev -d postgres:12.1-alpine
                            docker run --name cast_service -d -p 8002:8000 -e DATABASE_URI=postgresql://cast_db_username:cast_db_password@cast_db/cast_db_dev "$DOCKER_ID/$DOCKER_IMAGE_CAST_SERVICE:$DOCKER_TAG" sh -c "uvicorn app.main:app --reload --host 0.0.0.0 --port 8000"
                            docker network connect cast_network cast_db
                            docker network connect cast_network cast_service
                            docker exec -d cast_service uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
                            sleep 6
                            curl http://localhost:8002/api/v1/casts/docs
                            docker container rm cast_service -f
                            docker container rm cast_db -f
                            docker network rm cast_network -f
                        '''
                    }
                }
            }
        }

        stage('Docker Images Push') {
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")
            }
            parallel {
                stage('Push movie service') {
                    steps {
                        sh '''
                            docker login -u $DOCKER_ID -p $DOCKER_PASS
                            docker push $DOCKER_ID/$DOCKER_IMAGE_MOVIE_SERVICE:$DOCKER_TAG
                        '''
                    }
                }
                stage('Push cast service') {
                    steps {
                        sh '''
                            docker login -u $DOCKER_ID -p $DOCKER_PASS
                            docker push $DOCKER_ID/$DOCKER_IMAGE_CAST_SERVICE:$DOCKER_TAG
                        '''
                    }
                }
            }    
        }

        stage('Deploy environments') {
            environment {
                KUBECONFIG = credentials("config")
            }
            parallel {
                stage('Deploy dev') {
                    steps {
                         echo "BRANCH NAME ${env.BRANCH_NAME} "
                    }
                }
                stage('Deploy prod') {
                    when {
                        branch 'main'
                    }
                    steps {
                        timeout(time: 15, unit: "MINUTES") {
                            input message: 'Do you want to deploy in production ?', ok: 'Yes'
                        }                    
                        script {
                            echo "BRANCH NAME ${env.BRANCH_NAME} "
                        }
                    }
                }
            }    
        }

        
    }
}
