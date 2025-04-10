pipeline {
    agent any

    tools {
        maven 'Maven 3.8.6'
    }

    environment {
        IMAGE_NAME = "anushreegm12/java-microservice1"
        SONARQUBE_ENV = 'SonarQubeServer' 
    }
    
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Anushreegm12/sonarqube.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests=false'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQubeServer') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build & Push') {
            when {
                branch 'develop'
            }
            steps {
                sh 'docker build -t anushreegm12/java-microservice1 .'
                withDockerRegistry([credentialsId: 'docker-hub-credentials', url: '']) {
                    sh 'docker push anushreegm12/java-microservice1'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            when {
                branch 'develop'
            }
            steps {
                sh """
                kubectl apply -f k8s/deployment.yaml
                kubectl apply -f k8s/service.yaml
                """
            }
        }
    }
}
