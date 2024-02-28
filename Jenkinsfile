pipeline {
    agent any
    
    tools {
        jdk 'JDK'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-install'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git 'https://github.com/chennareddy5/DotNet-DEMO.git'
            }
        }
        
        stage('OWASP Dependency') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'OWASP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Trivy FS') {
            steps {
                sh "trivy fs ."
            }
        }
        
        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=Dotnet \
                        -Dsonar.projectKey=Dotnet \
                        -Dsonar.login=squ_133043c1cc9934edbd1d89edb85dd1c379b54c77'''
                }
            }
        } 
        
        stage('Docker build and tag') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'Docker-credentials', toolName: 'Docker') {
                        sh "make image"
                    }
                }
            }
        }
        
        stage("Trivy Image Scan") {
            steps {
                sh "trivy image chennareddy12/dotnet-demoapp "
            }
        } 
         
        stage('Docker Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'Docker-credentials', toolName: 'Docker') {
                        sh "make push"
                    }
                }
            }
        }
        
        stage('Deloy to container') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'Docker-credentials', toolName: 'Docker') {
                        sh "docker run -d -p 5000:5000  chennareddy12/dotnet-demoapp "
                        sh "docker ps -a"
                    }
                }
            }
        }
    }
}
