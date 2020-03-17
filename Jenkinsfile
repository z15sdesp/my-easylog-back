#!/usr/bin/env groovy

pipeline {
    agent any

    environment {
        SERVICE_NAME = "${env.PROJECT}${env.DEPLOY_ENV}"

        DOCKER_REGISTRY_URL = 'http://10.1.56.30:8083/'
        DOCKER_CREDENTIALS_ID = 'NEXUS_DEPLOY'
    }

    stages {
        stage("build") {
            steps {
                checkout scm
                sh 'mvn package -Dmaven.test.skip=true'
            }
        }

        stage('SonarQube analysis') {
            steps{
                // requires SonarQube Scanner 2.8+
                script {
                    scannerHome = tool 'sonar-icubator-scanner';
                    withSonarQubeEnv('sonar-icubator') {
                      sh "${scannerHome}/bin/sonar-scanner"
                    }
                }
            }
        }

        stage("package") {
            steps {
                script {
                    dockerImage = docker.build("${SERVICE_NAME}:${env.BUILD_NUMBER}", "-f Dockerfile .")
                }
            }
        }
        stage('Docker publish image') {
            steps {
                echo "publish docker image ..."
                script {
                    docker.withRegistry("${DOCKER_REGISTRY_URL}", DOCKER_CREDENTIALS_ID) {
                    dockerImage.push "${env.BUILD_NUMBER}"
                    dockerImage.push 'latest'
                    }
                }
            }
        }

    }
}
