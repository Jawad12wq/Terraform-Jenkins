pipeline {
    parameters {
        booleanParam(name: 'autoApprove', defaultValue: false, description: 'Automatically run apply after generating plan?')
    }

    agent any

    stages {
        stage('Checkout') {
            steps {
                script {
                    // Clone the repository into a specific directory
                    git branch: 'main', url: 'https://github.com/Jawad12wq/Terraform-Jenkins.git'
                }
            }
        }

        stage('Plan') {
            steps {
                withCredentials([
                    string(credentialsId: 'aws-access-key-id', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'aws-secret-access-key', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    script {
                        dir('terraform') {
                            // Terraform initialization
                            sh 'terraform init'
                            // Create Terraform plan
                            sh 'terraform plan -out=tfplan'
                            // Save human-readable plan to a file
                            sh 'terraform show -no-color tfplan > tfplan.txt'
                        }
                    }
                }
            }
        }

        stage('Approval') {
            when {
                not {
                    equals expected: true, actual: params.autoApprove
                }
            }
            steps {
                script {
                    // Read the plan file and display it for approval
                    def plan = readFile 'terraform/tfplan.txt'
                    input message: "Do you want to apply the plan?",
                    parameters: [text(name: 'Plan', description: 'Please review the plan', defaultValue: plan)]
                }
            }
        }

        stage('Apply') {
            steps {
                withCredentials([
                    string(credentialsId: 'aws-access-key-id', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'aws-secret-access-key', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    script {
                        dir('terraform') {
                            // Apply the Terraform plan
                            sh 'terraform apply -input=false tfplan'
                        }
                    }
                }
            }
        }
    }
}
