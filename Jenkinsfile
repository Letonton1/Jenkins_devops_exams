environment { // Declaration of environment variables
DOCKER_ID = "letonton" // replace this with your docker-id
DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
}

agent any // Jenkins will be able to select all available agents

stages {
  stage(' Docker Build'){ // docker build image stage
    steps {
      script {
      sh '''
        docker compose down 
        docker compose up -d
      sleep 20
      '''
      }
    }
  }
  stage('Test Acceptance'){ // we launch the curl command to validate that the container responds to the request
    steps {
      script {
      sh '''
      curl http://localhost:8080/api/v1/movies/docs
      curl http://localhost:8080/api/v1/casts/docs
      '''
      }
    }
  }

  stage('Docker Push'){ //we pass the built image to our docker hub account
    environment
    {
      DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
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
  stage('Deploiement en dev'){
    environment {
    KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
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
  post {
    failure {
        echo "This will run if the job failed"
        mail to: "vgcehill@gmail.com",
             subject: "${env.JOB_NAME} - Build # ${env.BUILD_ID} has failed",
             body: "For more info on the pipeline failure, check out the console output at ${env.BUILD_URL}"
    }
 }
}
