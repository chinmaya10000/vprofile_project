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
                APP_IMAGE_TAG = 'V1'
                DB_IMAGE_TAG = 'V1'
                WEB_IMAGE_TAG = 'V1'
            }
            steps {
                script {
                    echo "Build and Push Docker Images"
                    def dockerAppDir = 'Docker-files/app'
                    def dockerDbDir = 'Docker-files/db'
                    def dockerWebDir = 'Docker-files/web'

                    // Build and tag Docker images for app, db, and web using the defined tags
                    docker.build("chinmayapradhan/vprofileapp:${APP_IMAGE_TAG}", "-f ${dockerAppDir}Dockerfile ${dockerAppDir}")

                    // Push Docker images to a registry using the defined tags
                    docker.withRegistry
                }
            }
        }
    }
}