pipeline {
    agent any
    environment {
        SONAR_HOME = tool "Sonar"
        DOCKER_IMAGE = "abhinavnandekar/invoice-generator:latest"
        TARGET_URL = "192.168.144.128"
    }
    
    stages {
        stage ("Clone code from Github") {
            steps {
                git url: "https://github.com/abhinavnandekar27/invoice_generator.git", branch: "master"
            }
        }

        // start sonarqube server
        stage ("SAST Stage") {
            steps {
                // sh "docker start sonarqube-server"
                withSonarQubeEnv("Sonar") {
                    sh "$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=invoice -Dsonar.projectKey=invoice"
                }
            }
        }

        // stage("Dependency Check") {
        //     steps {
        //         dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'owasp'
        //         dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
        //     }
        // }
        
        stage("Build Docker Image") {
            steps {
                sh "docker build -t $DOCKER_IMAGE ."
            }
        }

        // To scan vulns in images,containers, file systems
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

        // Run docker-compose up on invoice-generator folder to manually deploy the project
        stage("OWASP ZAP Security Scan") {
            steps {
                sh """
                docker pull zaproxy/zap-stable
                docker run --rm -u root -v \$(pwd):/zap/wrk zaproxy/zap-stable zap-baseline.py -t http://192.168.144.128:80 -r zap_report.html || true
                """
            }
        }

        
    }
}
