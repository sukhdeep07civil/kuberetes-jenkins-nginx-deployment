pipeline {
    agent any 

    environment {
        IMAGE = 'ssm0712/kubernetes-jenkins-nginx-deployment'
    }

    stages{
        stage('Checkout scm'){
            steps{
                checkout scm
            }
        }

        stage('Set Version'){
            steps{
                script{
                    env.SHORT_SHA = bat(
                        script: '@echo off & git rev-parse --short HEAD',
                        returnStdout:true
                    ).trim()

                    env.VERSION = "${BUILD_NUMBER}-${env.SHORT_SHA}"
                    echo "VERSION: ${env.VERSION}"
                }
            }
        }

        stage('Build Docker Image'){
            steps{
                bat 'docker build -t %IMAGE%:%VERSION% .'
            }
        }

        stage('Tag Versions'){
            steps{
                bat """
                docker tag %IMAGE%:%VERSION% %IMAGE%:%BUILD_NUMBER%
                docker tag %IMAGE%:%VERSION% %IMAGE%:latest
                """
            }
        }

        stage('Push Images'){
            steps{
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]){
                    bat """
                    docker login -u %DOCKER_USER% -p %DOCKER_PASS%
                    docker push %IMAGE%:%VERSION%
                    docker push %IMAGE%:%BUILD_NUMBER%
                    docker push %IMAGE%:latest
                    """
                }
            }
        }

        stage('Deploy to kubernetes'){
            steps{
                bat """
                kubectl apply -f k8s/deployment.yaml
                kubectl apply -f k8s/service.yaml
                """
            }
        }
    }
}