pipeline {
    agent any

    tools {
        maven 'Maven 3.8.6'
    }

    environment {
        IMAGE_NAME = "anushreegm12/java-microservice1"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Anushreegm12/sonarqube.git'
            }
        }

        stage('Build and Test') {
            steps {
                sh 'mvn clean verify'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQubeServer') {
                    withCredentials([string(credentialsId: 'sonarqube-token', variable: 'SONAR_TOKEN')]) {
                        sh '''
                        echo "SonarQube Host: $SONAR_HOST_URL"
                        echo "Checking connectivity..."
                        curl -sSf $SONAR_HOST_URL/api/system/status || echo "SonarQube not reachable"
        
                        echo "Running Sonar Scanner..."
                        mvn -X sonar:sonar \
                            -Dsonar.projectKey=anushree-java-microservice \
                            -Dsonar.projectName="Anushree Java Microservice" \
                            -Dsonar.projectVersion=1.0 \
                            -Dsonar.sources=src/main/java \
                            -Dsonar.tests=src/test/java \
                            -Dsonar.login=$SONAR_TOKEN
                        '''
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
                sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
                withDockerRegistry([credentialsId: 'docker-hub-credentials', url: '']) {
                    sh 'docker push ${IMAGE_NAME}:${IMAGE_TAG}'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            when {
                branch 'develop'
            }
            steps {
                sh '''
                kubectl apply -f k8s/deployment.yaml
                kubectl apply -f k8s/service.yaml
                '''
            }
        }
    }
}
