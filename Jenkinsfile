pipeline {
    agent any
    environment {
        SONAR_HOME = tool "Sonar"
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
    }
}
