#!/usr/bin/env groovy

pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    stages {
        stage('Build and Package') {
            steps {
                script {
                    echo "Buils and Package WAR.."
                    sh 'mvn clean package'
                    sh 'cp -r target Docker-files/app/'
                }
            }
        }
        stage('Build and Push Images') {
            environment {
                IMAGE_TAG = 'V1'
            }
            steps {
                script {
                    echo "Build and Push Docker Images.."
                    def appImageName = "chinmayapradhan/vprofileapp"
                    def dbImageName = "chinmayapradhan/vprofiledb"
                    def webImageName = "chinmayapradhan/vprofileweb"

                    // Authenticate with Docker Hub using Jenkins credentials
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        // Build and push app Docker image with the specified tag
                        dir('Docker-files/app') {
                            sh "docker build -t ${appImageName}:${IMAGE_TAG} ."
                            sh "echo $PASS | docker login -u $USER --password-stdin"
                            sh "docker push ${appImageName}:${IMAGE_TAG}"
                        }
                        // Build and push db Docker image with the specified tag
                        dir('Docker-files/db') {
                            sh "docker build -t ${dbImageName}:${IMAGE_TAG} ."
                            sh "echo $PASS | docker login -u $USER --password-stdin"
                            sh "docker push ${dbImageName}:${IMAGE_TAG}"
                        }
                        // Build and push web Docker image with the specified tag
                        dir('Docker-files/web') {
                            sh "docker build -t ${webImageName}:${IMAGE_TAG} ."
                            sh "echo $PASS | docker login -u $USER --password-stdin"
                            sh "docker push ${webImageName}:${IMAGE_TAG}"
                        }
                    }
                    
                } 
            }
        }
        stage('deploy') {
            steps {
                script {
                    echo "Deploying Docker image to Azure VM"
                    def shellCmd = "docker-compose up -d"
                    withCredentials([sshUserPrivateKey(credentialsId: 'server-ssh-key', keyFileVariable: 'SSH_KEY')]) {
                        // Use the Azure VM's public IP address or DNS name
                        def azureVmIp = '20.193.157.22'
                        def sshUser = 'chinu'
                
                        // Deploy the Docker image to Azure VM using SSH
                        sh "echo \"${SSH_KEY}\" > prod-server_key.pem"
                        sh "chmod 600 prod-server_key.pem"
                        sh "ssh -o StrictHostKeyChecking=no -i prod-server_key.pem ${sshUser}@${azureVmIp} '${shellCmd}'"
                    }
                }
            }
        }
    }
}