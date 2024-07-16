pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
        
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
        
    }

    stages {
        stage('WorkSpace Clean') {
            steps {
                cleanWs()
            }
        }
        
         stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/kundan344/Zomato_App.git'
            }
        }
        
         stage('Sonar Qube Scanner') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=zomato \
                    -Dsonar.projectKey=zomato '''
            }
        }
         }
        
         stage('NPM Install') {
            steps {
                sh "npm install"
            }
        }
        
        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'dc'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        
        
         stage('Trivy file System Scan') {
            steps {
                sh "trivy fs --format table -o trivy-report.xml ."
            }
        }
        
        stage('Docker Image Build and Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-1', toolName: 'docker') {
                       sh "docker build -t kundankumar344/zomato:latest ."
                       sh "docker push kundankumar344/zomato:latest"
                   }
                    
                }
            }
        }
        
        stage('Trivy Docker Image Scan') {
            steps {
                sh "trivy image --format table -o trivy-image-report.xml kundankumar344/zomato:latest"
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-1', toolName: 'docker') {
                // Kill and remove the existing container if it exists
                sh """
                    if [ \$(docker ps -a -q -f name=zomato-app) ]; then
                        docker kill zomato-app || true
                        docker rm -f zomato-app || true
                    fi
                """
                // Run the new container
                sh "docker run --name zomato-app -p 3000:3000 kundankumar344/zomato:latest"
                    
                   }
                    
                }
            }
        }
        
        
       
    }
}