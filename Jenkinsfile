pipeline {
    agent any
    
    tools {
        maven 'mvn'
    }
    
    environment {
        SONAR_HOST_URL = 'http://3.27.71.222:9000/'
        SONAR_AUTH_TOKEN = credentials('sonarqube')
    }

    stages {
        stage('Git Checkout ') {
            steps {
                git url: 'https://github.com/AnushaAkkena/POC3', branch: 'main'
            }
        }
        
        stage('Code Compile') {
            steps {
                    sh "mvn compile"
            }
        }
        
        stage('Run Test Cases') {
            steps {
                    sh "mvn test"
            }
        }
        
        stage('Sonarqube Analysis') {
            steps {
                     sh '''
                    mvn sonar:sonar \
                      -Dsonar.projectKey=spring-boot-demo \
                      -Dsonar.host.url=$SONAR_HOST_URL \
                      -Dsonar.login=$SONAR_AUTH_TOKEN
                '''
    
                }
            }
        
        stage('OWASP Dependency Check') {
            steps {
                   dependencyCheck additionalArguments: '--scan ./   ', odcInstallation: 'DP'
                   dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Maven Build') {
            steps {
                    sh "mvn clean compile"
            }
        }
        
        stage('Docker Build & Push') {
            steps {
                   script {
                       withDockerRegistry(credentialsId: 'b289dc43-2ede-4bd0-95e8-75ca26100d8d', toolName: 'docker') {
                            sh "docker build -t webapp ."
                            sh "docker tag webapp adijaiswal/webapp:latest"
                            sh "docker push adijaiswal/webapp:latest "
                        }
                   } 
            }
        }
        
        stage('Docker Image scan') {
            steps {
                    sh "trivy image adijaiswal/webapp:latest "
            }
        }
        
    }
}
