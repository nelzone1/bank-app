pipeline {
    agent any

    tools {
        maven "maven"
    }

    environment {    
        DOCKERHUB_CREDENTIALS = credentials('Dockerhub_Cred')
    }

    stages {
        stage('SCM Checkout') {
            steps {
                echo 'Performing SCM Checkout'
                git 'https://github.com/Nelztacy/bank-app'
            }
        }

        stage('Application Build') {
            steps {
                echo 'Performing Maven Build'
                sh 'mvn -Dmaven.test.failure.ignore=true clean package'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker Image'
                sh "docker build -t nelzone/bankapp-eta-app:V${BUILD_NUMBER} ."
                sh 'docker image list'
                sh "docker tag nelzone/bankapp-eta-app:V${BUILD_NUMBER} nelzone/bankapp-eta-app:latest"
            }
        }

        stage('Approve - Push Image to DockerHub') {
            steps {
                script {
                    // Send an approval prompt
                    env.APPROVED_DEPLOY = input message: 'User input required. Choose "Yes" to continue | "Abort" to stop', 
                        ok: 'Yes', parameters: [choice(name: 'Proceed', choices: ['Yes', 'Abort'], description: 'Proceed with Docker image push?')]
                }
            }
        }

        stage('Login and Push to DockerHub') {
            when {
                expression {
                    env.APPROVED_DEPLOY == 'Yes'
                }
            }
            steps {
                script {
                    withDockerRegistry(credentialsId: 'Dockerhub_Cred') {
                        sh 'docker --version' // Optional: Verify Docker is available
                        sh "docker push nelzone/bankapp-eta-app:latest"
                    }
                }
            }
        }

        stage('Deploy Docker Container') {
            steps {
                script {
                    sh '''
                        # Stop and remove any existing container
                        docker stop bankapp-eta-app || true
                        docker rm bankapp-eta-app || true

                        # Run a new container from the latest image
                        docker run -d --name bankapp-eta-app -p 8081:8080 nelzone/bankapp-eta-app:latest
                    '''
                }
            }
        }
    }
}
