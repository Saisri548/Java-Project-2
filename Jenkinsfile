

pipeline {
    agent any

    tools {
        maven 'Maven-3.9.16'
    }

    environment {
        DOCKER_HOST = 'ec2-user@172.31.44.213'
        APP_NAME = 'my-java-app'
        DEPLOY_DIR = '/home/ec2-user/app-deploy'
    }

    stages {

        stage('Git Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/saisri548/my-java-app.git'
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Docker Build Preparation') {
            steps {
                sh '''
                    test -f target/my-java-app-1.0-SNAPSHOT.jar
                    test -f Dockerfile
                '''
            }
        }

        stage('Transfer Files') {
            steps {
                sshagent(['docker-server-ssh']) {
                    sh '''
                        ssh $DOCKER_HOST "mkdir -p $DEPLOY_DIR"

                        scp target/my-java-app-1.0-SNAPSHOT.jar \
                            Dockerfile \
                            $DOCKER_HOST:$DEPLOY_DIR/
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sshagent(['docker-server-ssh']) {
                    sh '''
                        ssh $DOCKER_HOST "
                            cd $DEPLOY_DIR &&
                            docker build -t $APP_NAME:$BUILD_NUMBER .
                        "
                    '''
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                sshagent(['docker-server-ssh']) {
                    sh '''
                        ssh $DOCKER_HOST "
                            docker rm -f $APP_NAME || true

                            docker run -d \
                                --name $APP_NAME \
                                --restart unless-stopped \
                                -p 8080:8080 \
                                $APP_NAME:$BUILD_NUMBER
                        "
                    '''
                }
            }
        }

        stage('Verify Container') {
            steps {
                sshagent(['docker-server-ssh']) {
                    sh '''
                        ssh $DOCKER_HOST "
                            docker ps --filter name=$APP_NAME
                        "
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Application deployed successfully!'
        }

        failure {
            echo 'Deployment failed. Check Console Output.'
        }
    }
}
