pipeline {
    agent any

    tools {
        maven "maven"
    }

    environment {    
        DOCKERHUB_CREDENTIALS = credentials('dockerloginid')
    }

    stages {
        stage('SCM_Checkout') {
            steps {
                echo 'Perform SCM Checkout'
                git 'https://github.com/Nelztacy/bank-app'
            }
        }

        stage('Application_Build') {
            steps {
                echo 'Perform Maven Build'
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
                    env.APPROVED_DEPLOY = input message: 'User input required. Choose "Yes" | "Abort"'
                }
            }
        }

        stage('Login to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerloginid', usernameVariable: 'DOCKERHUB_CREDENTIALS_USR', passwordVariable: 'DOCKERHUB_CREDENTIALS_PSW')]) {
                    sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                }
            }
        }

        stage('Publish to Docker Registry') {
            steps {
                sh "docker push nelzone/bankapp-eta-app:latest"
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                sshagent(['your-ssh-credential-id']) {
                    sh 'scp /var/lib/jenkins/workspace/bank-app/target/banking-0.0.1-SNAPSHOT.jar root@10.0.0.174:/usr/local/tomcat10/webapps/'
                }
            }
        }
    }
}
