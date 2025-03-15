pipeline {
    agent any
    
    tools{
        
        jdk 'jdk17'
    }
    
    environment {
        
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout ') {
            steps {
                git 'https://github.com/srikanth62/DotNet-ssk.git'
            }
        }
        
        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: ' --scan ./ ', odcInstallation: 'DC'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Trivy FS SCan') {
            steps {
                sh "trivy fs ."
            }
        }
        
        stage('Sonarqube Analysis') {
            steps {
                
                withSonarQubeEnv('sonar'){
                  sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=dotnet-demo \
                    -Dsonar.projectKey=dotnet-demo ''' 
               }
                
               
            }
        }
        
        stage('Docker Build & Tag') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "make image"
                    }
                }
            }
        }
        
        stage('Docker Push') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "make push"
                    }
                }
            }
        }
        
        stage('Docker Deploy') {
            steps {
                sh "docker run -d -p 5000:5000 adijaiswal/dotnet-demoapp"
            }
        }
        
        
    }
}
