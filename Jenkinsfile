pipeline {
    agent any
    tools{
        maven  'maven3'
    }
        
    stages {
        stage('Git Checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/praveenece431/shopping-kart.git']])
            }
        }
        
        stage('COMPILE') {
            steps {
                sh "mvn clean compile -DskipTests=true"
            }
        }
        
        stage('OWASP Scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'DP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Sonarqube') {
            environment {
            SONAR_URL = "http://20.10.12.79:9000"
           }
            steps {
            withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
             sh 'cd shopping-kart && mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL} -Dsonar.java.binaries=. -Dsonar.projectKey=Shopping-Cart'
             }
        }
        
        stage('Build') {
            steps {
                sh "mvn clean package -DskipTests=true"
            }
        }
        
        stage('Docker Build & Push') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        
                        sh "docker build -t shopping-cart -f docker/Dockerfile ."
                        sh "docker tag  shopping-cart praveen431ece/shopping-cart:latest"
                        sh "docker push praveen431ece/shopping-cart:latest"
                    }
                }
            }
        }
        
        
    }
}
