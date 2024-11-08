pipeline {
    agent none
    stages{
        stage('build') {
            agent{
                docker{
                    image 'maven:3.6.1-jdk-8-slim'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            steps {
                echo 'Compiling worker app'
                sh 'mvn compile'
            }
        }
        stage('test') {
            agent{
                docker{
                    image 'maven:3.6.1-jdk-8-slim'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            steps {
                echo 'Running unit tests on worker-app'
                sh 'mvn clean test'
            }
        }
        stage('package') {   
            when { 
                environment name: 'CHANGE_TARGET', value: 'main'
            }
            agent{
                docker{
                    image 'maven:3.6.1-jdk-8-slim'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            steps {
                echo 'Packaging worker app'
                sh 'mvn package -DskipTests'
                archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
            }
        }
        stage('docker-package-main') {
            when {
                environment name: 'CHANGE_TARGET', value: 'main'
            }
            agent any
            steps {
                echo 'Packaging vote app with docker'
                script {
                    docker.withRegistry('https://index.docker.io/v1/','docker-login') {
                        def workerImage = docker.build ("${env.GIT_URL.tokenize('/.')[-3]}/worker-app:${env.CHANGE_BRANCH.tokenize('/')[-1]}-${env.BUILD_ID}", '.')
                        workerImage.push()
                        workerImage.push('latest')
                    }
                }
            }
        }
    }
}