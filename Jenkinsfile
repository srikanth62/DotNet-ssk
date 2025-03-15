pipeline {
    agent any
    tools {
        jdk 'jdk21'
        maven 'maven3'  
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    
    stages {
        stage('Git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/srikanth62/DotNet-ssk.git'
            }
        }
        
        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }
        
        stage('Test') {
            steps {
                sh 'mvn test -DskipTests=true'
            }
        }
        
        stage('OWASP scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Trivy FS Scan') {
            steps {
                sh 'trivy fs .'
            }
        }
        
        stage('SonarQube analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''
                    $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectName=Ekart \
                    -Dsonar.projectKey=Ekart \
                    -Dsonar.java.binaries=.
                    '''
                }
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn package -DskipTests=true'
            }
        }
        
        stage('Build & Tag Docker Image') {
            steps {
                withDockerRegistry([credentialsId: 'docker-cred', url: 'https://index.docker.io/v1/']) {
                    sh 'docker build -t shopping-cart:dev -f docker/Dockerfile .'  
                    sh 'docker tag shopping-cart:dev ssistu92/shopping-cart:dev'
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                withDockerRegistry([credentialsId: 'docker-cred', url: 'https://index.docker.io/v1/']) {
                    sh 'docker push ssistu92/shopping-cart:dev'  
                }
            }
        }
        
        stage('Deploy in Docker Container') {
            steps {
                sh 'docker run -d -p 5000:5000 ssistu92/shopping-cart:dev'  
            }
        }
    }
}
