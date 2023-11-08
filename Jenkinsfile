#!/usr/bin/env groovy

pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    stages {
        stage('Increment Version') {
            steps {
                script {
                    echo "Increment the App version..."
                    sh 'mvn build-helper:parse-version versions:set \
                        -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} \
                        versions:commit'
                    def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
                    def version = matcher[0][1]
                    env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                }
            }
        }
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
            steps {
                script {
                    echo "Build and Push Docker Images.."

                    // Authenticate with Docker Hub using Jenkins credentials
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        // Build and push app Docker image with the specified tag
                        dir('Docker-files/app') {
                            sh "docker build -t chinmayapradhan/vprofileapp:${IMAGE_NAME} ."
                            sh "echo $PASS | docker login -u $USER --password-stdin"
                            sh "docker push chinmayapradhan/vprofileapp:${IMAGE_NAME}"
                        }
                        // Build and push db Docker image with the specified tag
                        dir('Docker-files/db') {
                            sh "docker build -t chinmayapradhan/vprofiledb:${IMAGE_NAME} ."
                            sh "echo $PASS | docker login -u $USER --password-stdin"
                            sh "docker push chinmayapradhan/vprofiledb:${IMAGE_NAME}"
                        }
                        // Build and push web Docker image with the specified tag
                        dir('Docker-files/web') {
                            sh "docker build -t chinmayapradhan/vprofileweb:${IMAGE_NAME} ."
                            sh "echo $PASS | docker login -u $USER --password-stdin"
                            sh "docker push chinmayapradhan/vprofileweb:${IMAGE_NAME}"
                        }
                    }
                    
                } 
            }
        }
        stage('deploy') {
            steps {
                script {
                    echo "Deploying Docker image to Azure VM"
                    def shellCmd = "bash ./server-cmds.sh ${IMAGE_NAME}"
                    sshagent(['server-ssh-key']) {
                        sh 'scp server-cmds.sh azureuser@172.172.163.83:/home/azureuser'
                        sh 'scp compose/docker-compose.yml azureuser@172.172.163.83:/home/azureuser'
                        sh "ssh -o StrictHostKeyChecking=no azureuser@172.172.163.83 '${shellCmd}'"
                    }
                }
            }
        }
        stage('Commit Version Update') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'git-hub-repo', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        sh 'git config --global user.email "jenkins@example.com"'
                        sh 'git config --global user.name "jenkins"'
                        sh "git remote set-url origin https://${USER}:${PASS}@github.com/chinmaya10000/vprofile_project.git"
                        sh 'git add .'
                        sh 'git commit -m "ci: version bump"'
                        sh 'git push origin HEAD:docker'
                    }
                }
            }
        }
    }
}
