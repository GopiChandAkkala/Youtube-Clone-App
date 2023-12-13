@Library('jenkins-shared-library') _
def ec2PublicIp
pipeline {
    agent any

    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
        AWS_ACCESS_KEY_ID = credentials('access-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
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
        stage('Npm'){          
            steps{
                sh 'npm install'
            }
        }
        //stage('OWASP Dependency Check'){          
          //  steps{
            //    dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
              //  dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            //}
        //} Taking too much time to build
        stage('Trivy File Scan'){          
            steps{
                trivyFilescan()
            }
        }
        stage("Docker Build Image") {            
            steps {
                script {
                    sh 'docker build --build-arg REACT_APP_RAPID_API_KEY=3ca50ddbfamshba5cfce1769b188p1640a3jsn81bb1542d16a -t akkalagopi/youtube-app-clone:latest .'
                }
            }
        }
        stage("Trivy Image Scan") {            
            steps {
                script {
                    sh """
                        trivy image akkalagopi/youtube-app-clone:latest > scan.txt
                        cat scan.txt                        
                        """
                }
            }
        }
        stage("Docker Image Push") {            
            steps {
                script {
                    sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                    sh "docker push akkalagopi/youtube-app-clone:latest"                         
                }
            }
        }
        stage('Terraform Init and Apply') {
            steps {
                dir('terraform/') {
                    script {
                        sh 'terraform init -no-color'
                        def tfOutput = sh(script: 'terraform apply -auto-approve', returnStdout: true).trim()
                    }
                }
            }
        }

        stage('Terraform get IP') {
            steps {
                dir('terraform/') {
                    script {
                        ec2PublicIp = sh(script: 'terraform output -json ec2_instance_public_ip', returnStdout: true).trim()

                        withEnv(['EC2_PUBLIC_IP=' + ec2PublicIp]) {
                            echo "EC2 Public IP: ${EC2_PUBLIC_IP}"
                        }
                    }
                }
            }
        }

        stage('Ansible playbook get IP') {
            steps {
                dir('ansible/') {
                    script {
                        echo "About to create inventory file"
                        writeFile file: 'inventory.ini', text: "my-ec2 ansible_host=${ec2PublicIp} ansible_user=ec2-user"
                        echo "Inventory file is created"
                    }
                }
            }
        }

        stage('Ansible play playbook') {
            steps {
                dir('ansible/') {
                    script {
                        withCredentials([sshUserPrivateKey(credentialsId: 'aws-keypair', keyFileVariable: 'SSH_PRIVATE_KEY')]) {
                            sh """
                                ansible-playbook -i inventory.ini  main.yml --private-key=\$SSH_PRIVATE_KEY --become
                            """
                        }
                    }
                }
            }
        }        
    }
    post {
        always {
            sh 'docker logout'
            sh "echo Youtube-APP is available at http://${ec2PublicIp}:8080"
        }
    }

}
