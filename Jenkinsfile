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
                    echo "deploy docker image to VM"
                    def dockerComposeCmd = "docker-compose up -d"
                    sshagent(['ec2-server-key']) {
                        sh "scp compose/docker-compose.yml chinu@20.193.157.22:/home/chinu"
                        sh "ssh -o StrictHostKeyChecking=no chinu@20.193.157.22:/home/chinu '${dockerComposeCmd}'"
                    }
                }
            }
        }
    }
}