pipeline {
    agent any
    triggers {
        githubPush()
    }
    environment {
        TF_CLI_ARGS = '-no-color'
    }

    stages {
        stage("Checkout") {
            steps {
                script {
                    echo 'Checking out code from Git'
                    checkout scm
                    echo 'Checked out code from Git'
                }
            }
        }

        stage("Lint Code") {
            steps {
                script {
                    echo 'Running Terraform Validation for lint'
                    withCredentials([aws(credentialsId: 'AWS_CRED', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                        sh 'terraform init'
                        echo 'Terraform initialized'
                        echo 'Running terraform validate'
                        sh 'terraform validate'
                        echo 'Terraform validation completed'
                    }
                }
            }
        }

        stage("Terraform Plan") {
            steps {
                script {
                    echo 'Running Terraform plan'
                    withCredentials([aws(credentialsId: 'AWS_CRED', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                        sh 'terraform plan -out=tfplan'
                        echo 'Terraform plan completed'
                    }
                }
            }
        }

        stage("Terraform Apply") {
            when {
                expression { env.BRANCH_NAME == 'main' }
                expression { currentBuild.rawBuild.getCause(hudson.model.Cause$UserIdCause) != null }
            }
            steps {
                script {
                    echo "Requesting manual approval to apply changes"
                    input message: 'Do you want to apply changes?', ok: 'Yes'
                    withCredentials([aws(credentialsId: 'AWS_CRED', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                        echo "Applying changes to infrastructure using Terraform"
                        if (fileExists('tfplan')) {
                            sh 'terraform apply tfplan'
                            echo "Terraform apply completed"
                        } else {
                            echo "Plan file does not exist, skipping apply."
                        }
                    }
                }
            }
        }

        stage("Terraform Destroy") {
            when {
                expression { env.BRANCH_NAME == 'main' }
                expression { currentBuild.rawBuild.getCause(hudson.model.Cause$UserIdCause) != null }
            }
            steps {
                script {
                    echo "Waiting for specified delay before destroying resources"
                    sleep time: 1, unit: 'HOURS'
                    echo "Now destroying the resources"
                    withCredentials([aws(credentialsId: 'AWS_CRED', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                        echo "Destroying infrastructure using Terraform"
                        sh 'terraform destroy -auto-approve'
                        echo "Terraform destroy completed"
                    }
                }
            }
        }
    }
    post {
        always {
            echo 'Cleaning up Terraform files'
            sh 'rm -rf .terraform'
            sh 'rm -rf tfplan'
            echo 'Cleaned up Terraform files'
        }
        failure {
            echo 'Terraform apply failed'
        }
    }
}
