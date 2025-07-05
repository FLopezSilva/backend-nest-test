pipeline {
    agent any
    // escenarios -> escenario -> pasos
    environment{
        NPM_CONFIG_CACHE= "${WORKSPACE}/.npm"
        dockerImageName = "us-west1-docker.pkg.dev/lab-agibiz/docker-repository"
        registry = "https://us-west1-docker.pkg.dev"
        registryCredentials = "gcp-registry"
    }
    stages{
        stage ("proceso de build y test") {
            agent {
                docker {
                    image 'node:22'
                    reuseNode true
                }
            }
            stages {
                stage("instalacion de dependencias"){
                    steps {
                        sh 'npm ci'
                    }
                }
                stage("instalacion de pruebas"){
                    steps {
                        sh 'npm run test:cov'
                    }
                }
                 stage("build de proyecto"){
                    steps {
                        sh 'npm run build'
                    }
                }
            }
        }
        stage ("build y push de imagen docker"){
            steps{
                script {
                    docker.withRegistry("${registry}", registryCredentials){
                    sh "docker build -t backend-nest-test-fls ."
                    sh "docker tag backend-nest-test-fls ${dockerImageName}/backend-nest-test-fls"
                    sh "docker tag backend-nest-test-fls ${dockerImageName}/backend-nest-test-fls:${BUILD_NUMBER}"
                    sh "docker push ${dockerImageName}/backend-nest-test-fls"
                    sh "docker push ${dockerImageName}/backend-nest-test-fls:${BUILD_NUMBER}"
                    }
                }
               
            }
        }
        stage ("actualiza kubernets"){
            agent {
                docker {
                    image 'alpine/k8s:1.30.2'
                    reuseNode true
                }
            }
            steps {
                withKubeConfig([credentialsId: 'gcp=kubeconfig']){
                    sh "kubectl -n lab-fls set image deployments/backend-nest-test-fls backend-nest-test-fls=${dockerImageName}/backend-nest-test-fls:${BUILD_NUMBER}"
                }
            }
        }
    }
}