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
                        sh "docker tag  shopping-cart praveen431ece/shopping-cart:${BUILD_NUMBER}"
                        sh "docker push praveen431ece/shopping-cart:${BUILD_NUMBER}"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
        environment {
        // Define environment variables
         DOCKER_REGISTRY = "docker.io" // Docker Hub registry URL
         KUBE_NAMESPACE = "default"
         KUBE_CLUSTER = "experimental-aks"
         }
        steps {
          script {
              sh '''
                 BUILD_NUMBER=${BUILD_NUMBER}
                 cd spring-boot-app
                 pwd && ls -l
                 sed -i "s/tagversion/${BUILD_NUMBER}/g" deploy-service.yaml
                 echo "** Print the certificate **"
                 echo "$kube"|base64 -d > /tmp/config
                 kubectl apply -f  deploy-service.yaml --kubeconfig /tmp/config
                 cat deploy-service.yaml
                 sleep 30
                 kubectl get pods -n cicd --kubeconfig /tmp/config
              '''
          }      
        }
    }
    }
}
