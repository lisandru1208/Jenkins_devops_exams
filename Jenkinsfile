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
  stage('Docker Build'){ // docker build image stage
    steps {
      script {
      sh '''
        docker rm -f movie-service cast-service bdd sweb 
        docker network rm mynet
        docker network create mynet

        docker build -t "$DOCKER_ID/$DOCKER_MS:latest" ./$DOCKER_MS
        docker tag "$DOCKER_ID/$DOCKER_MS:latest" "$DOCKER_ID/$DOCKER_MS:latest"

        docker build -t "$DOCKER_ID/$DOCKER_CAST:latest" ./$DOCKER_CAST
        docker tag "$DOCKER_ID/$DOCKER_CAST:latest" "$DOCKER_ID/$DOCKER_CAST:latest"

        docker pull $BDD:12.1-alpine
        docker tag $BDD:12.1-alpine "$DOCKER_ID/$BDD:12.1-alpine"
        docker tag $BDD:12.1-alpine "$DOCKER_ID/$BDD:latest"
        
        docker pull $NGINX:latest
        docker tag $NGINX:latest "$DOCKER_ID/$NGINX:latest"
        docker tag $NGINX:latest "$DOCKER_ID/$NGINX:latest"
      sleep 6
      '''
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
        steps {
            script {
                withCredentials([string(credentialsId: 'DOCKER_HUB_PASS', variable: 'DOCKER_PASS')]) {
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
    } 
/*   stage('Deploiement en dev'){
    environment {
    KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
    }
    steps {
      script {
      sh '''
      rm -Rf .kube
      mkdir .kube
      ls
      cat $KUBECONFIG > .kube/config
      cp fastapi/values.yaml values.yml
      cat values.yml
      sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
      helm upgrade --install app fastapi --values=values.yml --namespace dev
      '''
      }
    }
  }
  stage('Deploiement en staging'){
    environment {
    KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
    }
    steps {
      script {
      sh '''
      rm -Rf .kube
      mkdir .kube
      ls
      cat $KUBECONFIG > .kube/config
      cp fastapi/values.yaml values.yml
      cat values.yml
      sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
      helm upgrade --install app fastapi --values=values.yml --namespace staging
      '''
      }
    }
  }
  stage('Deploiement en prod'){
    environment {
      KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
      }
      steps {
        // Create an Approval Button with a timeout of 15minutes.
        // this require a manuel validation in order to deploy on production environment
        timeout(time: 15, unit: "MINUTES") {
          input message: 'Do you want to deploy in production ?', ok: 'Yes'
        }
        script {
          sh '''
          rm -Rf .kube
          mkdir .kube
          ls
          cat $KUBECONFIG > .kube/config
          cp fastapi/values.yaml values.yml
          cat values.yml
          sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
          helm upgrade --install app fastapi --values=values.yml --namespace prod
          '''
        }
      }
    } */
  }
}