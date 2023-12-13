@Library('jenkins-shared-library') _

pipeline {
    agent any

    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    
    stages{
        stage("Clean WorkSpace") {            
            steps {
                script {
                    clearWorkspace()
                }
            }
        }

        stage("Git Checkout") {            
            steps {
                script {
                    gitCheckout(
                        branch: "main",
                        url: "https://github.com/GopiChandAkkala/Youtube-Clone-App.git"
                    )
                }
            }
        }
        stage("Sonaqube Analysis") {            
            steps {
                script {
                    sonarqubeAnalysis()
                }
            }
        }
        stage("Quality Gate Status: Sonarqube") {            
            steps {
                script {
                    def sonarqubecredentialsId = 'sonarqube-api'
                    qualityGateStatus(sonarqubecredentialsId)
                }
            }
        }

    }

}
