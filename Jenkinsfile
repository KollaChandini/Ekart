pipeline {
    agent any
    
    tools{
        jdk 'jdk17'
        maven 'maven3'
    }
    
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/KollaChandini/Ekart.git'
            }
        }
        
        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }
        
        stage('Unit Test') {
            steps {
               sh 'mvn test -DskipTests=true'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                sh '''
                    $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectKey=EKART \
                    -Dsonar.projectName=EKART \
                    -Dsonar.java.binaries=. '''
                }
            }
        }
        
        stage('OWASP') {
            steps {
                dependencyCheck additionalArguments: ' --scan ./ ', odcInstallation: 'owasp-dc'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn package -DskipTests=true'
            }
        }
        
        stage('Deploy to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-maven', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                sh 'mvn deploy -DskipTests=true'
                }
            }
        }
        
        stage('Docker Build & Tag Image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh 'docker build -t kollachandini/ekart:latest -f docker/Dockerfile .'
                    }
                }
            }
        }
        
        stage('Trivy Scan') {
            steps {
                sh 'trivy image kollachandini/ekart:latest > trivy-report.txt'
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh 'docker push kollachandini/ekart:latest '
                    }
                }
            }
        }
        
        stage('Kubernetes Deploy') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.xx.xx.1:6443') {
                sh 'kubectl apply -f deploymentservice.yml -n webapps'
                sh 'kubectl get svc -n webapps'
                }
            }
        }
    }
}
