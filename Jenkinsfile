pipeline {
    agent any

    environment {
        CONTAINER_NAME = "python-obfus-poc-container"
        PORT = "5000"
	IMAGE_NAME = 'prashanth2paramaah/python-obfus-poc:latest'
        VAULT_CRED_ID = 'vault-approle-creds'
        VAULT_ADDR    = 'http://18.207.117.176:8200'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/ParamaahIT/python-obfuscated.git'
            }
        }
        stage('Docker Login') {
            steps {
              script {
                withVault(
                  vaultUrl: "${env.VAULT_ADDR}",
                  vaultCredentialId: "${env.VAULT_CRED_ID}",
                  vaultSecrets: [
                   [
                     path: 'secret/docker-credentials',    
                     engineVersion: 2,
                     secretValues: [
                       [envVar: 'DOCKER_USERNAME', vaultKey: 'username'],
                       [envVar: 'DOCKER_PASSWORD', vaultKey: 'password']
                     ]
                   ]
                 ]
               ) {
                 sh '''
                   echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin

                 '''
               }
             }
           }
         }

        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                        docker build -t $IMAGE_NAME .
                    """
                }
            }
        }
        
       stage('Push Docker Image') {
         steps {
           sh '''
              docker push $IMAGE_NAME
      
            '''
          }
        }
       
      stage('Pull Docker Image') {
        steps {
          sh '''
            docker pull $IMAGE_NAME
            docker logout
      
           '''
          }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    sh """
                        docker rm -f $CONTAINER_NAME || true
                        docker run -d --name $CONTAINER_NAME -p $PORT:$PORT $IMAGE_NAME
                    """
                }
            }
        }
    }


}

