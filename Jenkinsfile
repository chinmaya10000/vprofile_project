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
    }
}