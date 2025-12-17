pipeline {
    agent any
    
    tools {
        maven 'Maven-3.8.7'
        jdk 'JDK-17'
    }
    
    environment {
        SONAR_HOST_URL = 'http://192.168.3.128:9000'
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
    }
    
    post {
        success {
            echo 'Pipeline completed successfully!'
            echo 'Coverage report available at: target/site/jacoco/index.html'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
