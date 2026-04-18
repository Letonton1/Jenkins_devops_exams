pipeline {
    environment {
        DOCKER_ID  = "letonton"
        DOCKER_TAG = "v.${BUILD_ID}.0"
    }
    agent any

    stages {

        stage('Docker Build') {
            steps {
                script {
                    sh '''
                        docker compose down || true
                        docker compose up -d
                        sleep 20
                    '''
                }
            }
        }

        stage('Test Acceptance') {
            steps {
                script {
                    sh '''
                        curl -f http://localhost:8080/api/v1/movies/docs
                        curl -f http://localhost:8080/api/v1/casts/docs
                    '''
                }
            }
        }

        stage('Docker Push') {
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")
            }
            steps {
                script {
                    sh '''
                        docker login -u $DOCKER_ID -p $DOCKER_PASS
                        docker tag jenkins_devops_exams-movie_service $DOCKER_ID/movie-service:$DOCKER_TAG
                        docker tag jenkins_devops_exams-cast_service  $DOCKER_ID/cast-service:$DOCKER_TAG
                        docker push $DOCKER_ID/movie-service:$DOCKER_TAG
                        docker push $DOCKER_ID/cast-service:$DOCKER_TAG
                    '''
                }
            }
        }

        stage('Deploiement en dev') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                        rm -Rf .kube
                        mkdir .kube
                        cat $KUBECONFIG > .kube/config
                        cp charts/values.yaml values.yml
                        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                        sed -i "s+repository.*+repository: ${DOCKER_ID}/movie-service+g" values.yml
                        helm upgrade --install movie-service --values=values.yml --namespace dev ./charts

                        cp charts/values.yaml values.yml
                        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                        sed -i "s+repository.*+repository: ${DOCKER_ID}/cast-service+g" values.yml
                        helm upgrade --install cast-service --values=values.yml --namespace dev ./charts
                    '''
                }
            }
        }

        stage('Deploiement en QA') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                        rm -Rf .kube
                        mkdir .kube
                        cat $KUBECONFIG > .kube/config
                        cp charts/values.yaml values.yml
                        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                        sed -i "s+repository.*+repository: ${DOCKER_ID}/movie-service+g" values.yml
                        helm upgrade --install movie-service --values=values.yml --namespace qa ./charts

                        cp charts/values.yaml values.yml
                        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                        sed -i "s+repository.*+repository: ${DOCKER_ID}/cast-service+g" values.yml
                        helm upgrade --install cast-service --values=values.yml --namespace qa ./charts
                    '''
                }
            }
        }

        stage('Deploiement en staging') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                        rm -Rf .kube
                        mkdir .kube
                        cat $KUBECONFIG > .kube/config
                        cp charts/values.yaml values.yml
                        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                        sed -i "s+repository.*+repository: ${DOCKER_ID}/movie-service+g" values.yml
                        helm upgrade --install movie-service --values=values.yml --namespace staging ./charts

                        cp charts/values.yaml values.yml
                        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                        sed -i "s+repository.*+repository: ${DOCKER_ID}/cast-service+g" values.yml
                        helm upgrade --install cast-service --values=values.yml --namespace staging ./charts
                    '''
                }
            }
        }

        stage('Deploiement en prod') {
            when {
                branch 'master'
            }
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    input message: 'Déployer en production ?', ok: 'Oui, déployer'
                }
                script {
                    sh '''
                        rm -Rf .kube
                        mkdir .kube
                        cat $KUBECONFIG > .kube/config
                        cp charts/values.yaml values.yml
                        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                        sed -i "s+repository.*+repository: ${DOCKER_ID}/movie-service+g" values.yml
                        helm upgrade --install movie-service --values=values.yml --namespace prod ./charts

                        cp charts/values.yaml values.yml
                        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                        sed -i "s+repository.*+repository: ${DOCKER_ID}/cast-service+g" values.yml
                        helm upgrade --install cast-service --values=values.yml --namespace prod ./charts
                    '''
                }
            }
        }
    }
}
