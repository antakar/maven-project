pipeline {
    agent any
    stages{
        stage('Build'){
            steps{
                sh 'mvn -B clean package'
                sh "docker build . -t tomcatwebapp:${env.BUILD_ID}"
            }
        }
    }
}