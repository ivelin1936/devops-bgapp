pipeline 
{
  environment 
  {
    DOCKERHUB_CREDENTIALS=credentials('docker-hub')
    PROJECT_VOLUME="\/home\/vagrant\/web"
    VOLUME="/home/vagrant/web"
    WEB_FOLDER="web"
    REPO_URL="http://192.168.56.12:3000/douser/devops-bgapp.git"
    WEB_IMG="m5_web"
    DB_IMG="m5_db"
    PRJ_ENV_FILE=".env"
    COMPOSE_DEV="docker-compose-dev.yaml"
    COMPOSE_PROD="docker-compose-prod.yaml"
  }

  agent 
  {
    label 'docker-node'
  }

  stages 
  {
    stage('Clone Repo')
    {
      steps
      {
        git branch: 'master', url: '$REPO_URL'
      }
    }

    stage('Prepare project environments')
    {
      steps
      {
        sh '
            if [[ -f "$PRJ_ENV_FILE" ]]; then
                sed -i "s/^PROJECT_ROOT=.*$/PROJECT_ROOT=$PROJECT_VOLUME/g" $PRJ_ENV_FILE || true
                sed -i "s/^DO_HUB_USR=.*$/DO_HUB_USR=$DOCKERHUB_CREDENTIALS_USR/g" $PRJ_ENV_FILE || true
                sed -i "s/^WEB_IMG=.*$/WEB_IMG=$WEB_IMG/g" $PRJ_ENV_FILE || true
                sed -i "s/^DB_IMG=.*$/DB_IMG=$DB_IMG/g" $PRJ_ENV_FILE || true
            fi
        '
      }
    }

    stage('Prepare volumes')
    {
      steps
      {
        sh 'sudo rm -r $VOLUME || true'
        sh 'sudo cp -r $WEB_FOLDER $VOLUME'
      }
    }

    stage('Deploy on Staging')
    {
      steps
      {
        sh 'docker-compose -f $COMPOSE_DEV down || true'
        sh 'docker-compose -f $COMPOSE_DEV up -d'
      }
    }

    stage('Test')
    {
      steps
      {
        script 
        {
          echo 'Test #1 - reachability'
          sh 'echo $(curl --write-out "%{http_code}" --silent --output /dev/null http://localhost:8080) | grep 200'
          
          echo 'Test if DB works - check Plovdiv is displayed'
        //   sh 'echo $(curl -silent http://localhost:8080) | grep -oh "\\w*Пловдив\\w*"'
        }
      }
    }

    stage('Stop app and remove from Staging')
    {
      steps
      {
        sh 'docker-compose -f $COMPOSE_DEV down || true'
        // sh 'docker container stop $(docker ps -a -q --filter "ancestor=$DOCKERHUB_CREDENTIALS_USR/$WEB_IMG") || true'
        // sh 'docker rm $(docker ps -a -q --filter "ancestor=$DOCKERHUB_CREDENTIALS_USR/$WEB_IMG") || true'
        // sh 'docker container stop $(docker ps -a -q --filter "ancestor=$DOCKERHUB_CREDENTIALS_USR/$DB_IMG") || true'
        // sh 'docker rm $(docker ps -a -q --filter "ancestor=$DOCKERHUB_CREDENTIALS_USR/$DB_IMG")) || true'
      }
    }

    stage('Publishing the images to Docker Hub') 
    {
      steps 
      {
        sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
        sh 'docker image tag $(basename $(pwd))_web $DOCKERHUB_CREDENTIALS_USR/$WEB_IMG'
        sh 'docker push $DOCKERHUB_CREDENTIALS_USR/$WEB_IMG'
        sh 'docker image tag $(basename $(pwd))_db $DOCKERHUB_CREDENTIALS_USR/$DB_IMG'
        sh 'docker push $DOCKERHUB_CREDENTIALS_USR/$DB_IMG'
      }
    }

    stage('Remove images from Staging')
    {
      steps
      {
        sh 'docker-compose -f $COMPOSE_DEV down --rmi all || true'
      }
    }

    stage('Deploy App on Prod')
    {
      steps
      {
        sh 'docker image rm $DOCKERHUB_CREDENTIALS_USR/$WEB_IMG || true'
        sh 'docker image rm $DOCKERHUB_CREDENTIALS_USR/$DB_IMG || true'
          
        sh 'docker-compose -f $COMPOSE_PROD down || true'
        sh 'docker-compose -f $COMPOSE_PROD up -d'
      }
    }
  }

  post 
  { 
    always 
    { 
      cleanWs()
    }
  }
} 
