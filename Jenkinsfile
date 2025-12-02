pipeline {
    agent any
    
    tools {
        maven 'Maven-3.8.7'
        jdk 'JDK-17'
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
        
        stage('Test') {
            steps {
                echo 'Running unit tests...'
                sh 'mvn test'
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
