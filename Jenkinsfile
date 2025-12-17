pipeline {
    agent any
    
    tools {
        //jdk 'jdk11'
        maven 'maven3'
    }
    
    environment {
        SONAR_HOST_URL = 'http://13.210.236.226:9000/'
        SONAR_AUTH_TOKEN = credentials('sonarqube')
    }

    stages {
        stage('Git Checkout ') {
            steps {
                git url: 'https://github.com/AnushaAkkena/POC-6.git', branch: 'main'
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
            environment {
                NVD_API_KEY = credentials('nvd-api-key')
            }
            steps {
                sh """
                    mvn org.owasp:dependency-check-maven:9.0.9:check \
                      -Dnvd.api.key=\${NVD_API_KEY} \
                      -Dnvd.api.delay=6000 \
                      -Dnvd.api.maxRetryCount=15 \
                      -DautoUpdate=false \
                      -DfailOnError=false
                """
            }
            post {
                always {
                    script {
                        if (fileExists('target/dependency-check-report.html')) {
                            archiveArtifacts artifacts: 'target/dependency-check-report.html',
                                             fingerprint: true
                        } else {
                            echo 'Dependency-Check report not generated'
                        }
                    }
                }
            }
        }
        
        stage('Maven Build') {
            steps {
                    sh "mvn clean package"
            }
        }
        
        stage('Docker Build & Push') {
            steps {
                   script {
                       withDockerRegistry(credentialsId: 'dockerhub', toolName: 'docker') {
                            sh "docker build -t webapp ."
                            sh "docker tag webapp anushaakkena/webapp:latest"
                            sh "docker push anushaakkena/webapp:latest "
                        }
                   } 
            }
        }
        
        stage('Docker Image scan') {
            steps {
                    sh "trivy image anushaakkena/webapp:latest "
            }
        }
        
    }
}
