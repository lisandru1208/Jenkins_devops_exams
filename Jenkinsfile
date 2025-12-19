pipeline {
    environment { // Declaration of environment variables
      DOCKER_ID = "lisandru1208"
      DOCKER_MS = "movie-service"
      DOCKER_CAST = "cast-service"
      BDD = "postgres"
      NGINX = "nginx"
    }
agent any // Jenkins will be able to select all available agents
    stages {
      stage('Docker Build') {
        steps {
          script {
            movieService = docker.build("$DOCKER_ID/$DOCKER_MS:latest", "./DOCKER_MS")
            castService = docker.build("$DOCKER_ID/$DOCKER_CAST:latest", "./DOCKER_CAST")
            bddService = docker.pull("$NGINX:latest", "$DOCKER_ID/$NGINX:latest")
            nginxService = docker.pull("$BDD:12.1-alpine", "$DOCKER_ID/$BDD:latest")
          }
        }  
      }
      stage('Docker run'){ // run container from our builded image
        steps {
          script {
          sh '''
          docker run -d -p 8001:80 --network mynet --name movie-service $DOCKER_ID/$DOCKER_MS:latest
          docker run -d -p 8002:80 --network mynet --name cast-service $DOCKER_ID/$DOCKER_CAST:latest
          docker run -d -p 5432:5432 --network mynet --name bdd $DOCKER_ID/$BDD:latest
          docker run -d -p 8081:80 --network mynet --name sweb $DOCKER_ID/$NGINX:latest
          sleep 10
          '''
          }
        }
      }
      stage('Test Acceptance'){ // we launch the curl command to validate that the container responds to the request
        steps {
          script {
          sh '''
          curl localhost
          '''
          }
        }
      }
      stage('Docker Push') {
        environment {
          DOCKER_PASS = credentials('DOCKER_HUB_PASS') //variable secretS
        }
        steps {
            script {
                    sh '''
                        docker login -u $DOCKER_ID -p $DOCKER_PASS
                        docker push $DOCKER_ID/$DOCKER_MS:$latest --quiet=false --progress=plain
                        docker push $DOCKER_ID/$DOCKER_CAST:$latest --quiet=false --progress=plain
                        docker push $DOCKER_ID/$BDD:$latest --quiet=false --progress=plain
                        docker push $DOCKER_ID/$NGINX:$latest --quiet=false --progress=plain
                    '''
            }
        }
      }
      stage('Push image') {
        // Push sécurisé via credentials Jenkins
          steps {
            script {
              docker.withRegistry('', '$DOCKER_PASS') {
                  movieService.push("latest")        
                  castService.push("latest")               
                  bddService.push("latest")                      
                  nginxService.push("latest")               
              }  
            }
          }   
      }

    }
    
}