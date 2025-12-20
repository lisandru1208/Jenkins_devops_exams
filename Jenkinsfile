pipeline {
    agent any
    environment {
        DOCKER_ID    = "lisandru1208"
        DOCKER_MS    = "movie-service"
        DOCKER_CAST  = "cast-service"
        BDD          = "postgres"
        NGINX        = "nginx"
    }
    stages {
        stage('Docker Build') {
            steps {
                script {
                    sh '''
                        docker rm -f movie-service cast-service bdd sweb || true
                        docker network rm mynet || true
                        docker network create mynet
                    '''
                    // Construire les images locales
                    movieService = docker.build("$DOCKER_ID/$DOCKER_MS:latest", "./$DOCKER_MS")
                    castService  = docker.build("$DOCKER_ID/$DOCKER_CAST:latest", "./$DOCKER_CAST")
                    
                    // Récupérer les images existantes
                    bddService   = docker.image("$BDD:12.1-alpine")
                    bddService.pull()
                    
                    nginxService = docker.image("$NGINX:latest")
                    nginxService.pull()
                }
            }
        }

        stage('Docker Run') {
            steps {
                script {
                    sh """
                    docker run -d -p 8001:80 --network mynet --name movie-service $DOCKER_ID/$DOCKER_MS:latest
                    docker run -d -p 8002:80 --network mynet --name cast-service $DOCKER_ID/$DOCKER_CAST:latest
                    docker run -d -p 5432:5432 --network mynet --name bdd $BDD:12.1-alpine
                    docker run -d -p 8081:80 --network mynet --name sweb $NGINX:latest
                    sleep 10
                    """
                }
            }
        }

        stage('Test Acceptance') {
            steps {
                script {
                    sh 'curl localhost'
                }
            }
        }

        stage('Docker Push') {
            steps {
              script {
                docker.withRegistry('', 'DOCKER_HUB_PASS') {
                    movieService.push("latest")
                    castService.push("latest")
                }
              }
            }
        }
          stage('Deploiement en dev'){
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
                cp charts/values.yaml values.yaml
                cat values.yaml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yaml
                helm upgrade --install app charts --values=values.yml --namespace dev
                '''
                }
            }
        }
                  stage('Deploiement en QA'){
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
                cp charts/values.yaml values.yaml
                cat values.yaml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yaml
                helm upgrade --install app charts --values=values.yml --namespace qa
                '''
                }
            }
        }
                  stage('Deploiement en staging '){
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
                cp charts/values.yaml values.yaml
                cat values.yaml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yaml
                helm upgrade --install app charts --values=values.yml --namespace staging
                '''
                }
            }
        }
                  stage('Deploiement en prod'){
            environment {
                KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
            }
            steps {
                timeout(time: 15, unit: "MINUTES") {
                   input message: 'Voulez vous deployer en production ?', ok: 'Yes'
                }
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp charts/values.yaml values.yaml
                cat values.yaml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yaml
                helm upgrade --install app charts --values=values.yml --namespace prod
                '''
                }
            }
        }
    }
}
