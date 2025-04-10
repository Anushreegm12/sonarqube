pipeline {
    agent any

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

    stages {
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
                withCredentials([string(credentialsId: 'SonarQubeServer', variable: 'SONAR_TOKEN')]) {
                    sh 'mvn sonar:sonar -Dsonar.login=$SONAR_TOKEN'
                    }
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
            }
        }
        stage('Push to DockerHub') {
            steps {
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
