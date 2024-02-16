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
                        -Dsonar.login=squ_fa33bb9c1483ffaa0180f028a48dbee6412bef73'''
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
