pipeline {
    agent any
    
    tools {
        maven 'Maven-3.8.7'
        jdk 'JDK-17'
    }
    
    environment {
        SONAR_HOST_URL = 'http://192.168.3.128:9000'
        DOCKER_IMAGE = 'jvvfer/student-management'
        DOCKER_TAG = '1.0'
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                echo 'Cloning repository from GitHub...'
                git branch: 'main', url: 'https://github.com/jvvfer/student-management.git'
            }
        }
        
        stage('Build') {
            steps {
                echo 'Compiling the project...'
                sh 'mvn clean compile'
            }
        }
        
        stage('Test & Coverage') {
            steps {
                echo 'Running unit tests with coverage...'
                sh 'mvn test jacoco:report'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                echo 'Analyzing code quality with SonarQube...'
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        mvn sonar:sonar \
                        -Dsonar.projectKey=student-management \
                        -Dsonar.projectName=student-management \
                        -Dsonar.java.coveragePlugin=jacoco \
                        -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
                    '''
                }
            }
        }
        
        stage('Package') {
            steps {
                echo 'Creating JAR package...'
                sh 'mvn package -DskipTests'
            }
        }
        
        stage('Docker Build') {
            steps {
                echo 'Building Docker image...'
                sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
            }
        }
        
        stage('Docker Push') {
            steps {
                echo 'Pushing Docker image to Docker Hub...'
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
                    sh "docker push ${DOCKER_IMAGE}:latest"
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploying to Kubernetes cluster...'
                sh '''
                    kubectl apply -f k8s/mysql-deployment.yaml
                    kubectl apply -f k8s/spring-deployment.yaml
                    kubectl rollout status deployment/spring-app -n devops
                '''
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline completed successfully!'
            echo "Docker image available at: ${DOCKER_IMAGE}:${DOCKER_TAG}"
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
