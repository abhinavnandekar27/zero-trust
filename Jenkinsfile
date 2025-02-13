pipeline {
    agent any
    environment {
        SONAR_HOME = tool "Sonar"
        DOCKER_IMAGE = "abhinavnandekar/invoice-generator:latest"
    }
    
    stages {
        stage ("Clone code from Github") {
            steps {
                git url: "https://github.com/abhinavnandekar27/invoice_generator.git", branch: "master"
            }
        }
        
        stage ("SAST Stage") {
            steps {
                withSonarQubeEnv("Sonar") {
                    sh "$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=invoice -Dsonar.projectKey=invoice"
                }
            }
        }
        
        stage("Build Docker Image") {
            steps {
                sh "docker build -t $DOCKER_IMAGE ."
            }
        }

        stage("Security Scan with Trivy") {
            steps {
                sh "trivy image --exit-code 1 --severity HIGH,CRITICAL $DOCKER_IMAGE || echo 'Vulnerabilities detected'"
            }
        }
        
        stage("Login to Docker Hub") {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                }
            }
        }

        stage("Push Docker Image to Docker Hub") {
            steps {
                sh "docker push $DOCKER_IMAGE"
            }
        }
    }
}
