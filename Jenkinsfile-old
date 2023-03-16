pipeline {
    
    agent { dockerfile true }

 //   environment {
 //       registry = "068643504245.dkr.ecr.us-east-1.amazonaws.com/api-repo"
 //   }
    stages {
        //Loading variables
        stage('Load Variables') {
            steps {
                script {
                    //make sure that file exists on this node
                    def constants = load 'Variables'
                    registry = constants.registryECR
                    branch = constants.branchName
                    gitURL = constants.gitURL
                    echo branch
                    echo registry
                    echo gitURL
                }
            }
        }
        //wanted to run these 2 below commands in shell - having permission issues
        //usermod -a -G jenkins jenkins
        //chmod 777 /var/run/docker.sock
        stage ('Print Variables') {
            steps {
                script {
                    echo branch
                    echo registry
                    echo gitURL
                }
            }
        }
        stage ('Checkout') {
            steps {
                checkout scmGit(branches: [[name: branch]], extensions: [], userRemoteConfigs: [[url: gitURL]])
            }
        }
        stage ('Docker Image Build') {
            steps {
                script {
                    dockerImage = docker.build registry
                }
            }
        }
        //This code pushes the image to AWS ECR
        stage ('Docker Push') {
            steps {
                script {
                    sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 068643504245.dkr.ecr.us-east-1.amazonaws.com'
                    sh 'docker push 068643504245.dkr.ecr.us-east-1.amazonaws.com/api-repo:latest'
                }
            }
        }
        //Stopping Docker containers for cleaner Docker run
        stage ('stop previous containers') {
            steps {
                sh 'docker ps -f name=mypythonContainer -q | xargs --no-run-if-empty docker container stop'
                sh 'docker container ls -a -fname=mypythonContainer -q | xargs -r docker container rm'
            }
        }
        //This will run the Docker container
        stage ('Docker Run') {
            steps {
                script {
                    sh 'docker run -d -p 8096:5000 --rm --name mypythonContainer 068643504245.dkr.ecr.us-east-1.amazonaws.com/api-repo:latest'
                }
            }
        }
    }
}
