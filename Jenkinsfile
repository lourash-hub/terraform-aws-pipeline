pipeline {
    agent any // Defines that this pipeline can run on any available agent.

    environment {
        TF_CLI_ARGS = '-no-color' // Set environment variable to disable color in Terraform CLI output for better readability in Jenkins logs.
    }

    stages {
        // Stage to validate and lint Terraform configurations
        stage('Lint Code') {
            steps {
                script {
                    echo 'Starting Terraform code linting...' // Log the start of linting process.
                    sh 'terraform init' // Initialize the Terraform, required before validation.
                    sh 'terraform validate' // Validate the Terraform files for syntax and logical errors.
                    echo 'Terraform code linting completed successfully.' // Log the successful completion of linting process.
                }
            }
        }

        // Stage to checkout the source code from the SCM (e.g., Git repository)
        stage('Checkout') {
            steps {
                script {
                    echo 'Checking out the repository...' // Log the start of the checkout process.
                    checkout scm // Check out the source code from the repository as defined in the pipeline configuration.
                    echo 'Repository checked out.' // Log the successful completion of checkout process.
                }
            }
        }

        // Stage to plan Terraform deployment
        stage('Terraform Plan') {
            steps {
                script {
                    withCredentials([aws(credentialsId: 'AWS_CRED', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                        echo 'Initializing Terraform...' // Log the start of Terraform initialization.
                        sh 'terraform init' // Initialize Terraform with the provided credentials.
                        echo 'Running Terraform plan...' // Log the start of Terraform planning process.
                        sh 'terraform plan -out=tfplan' // Execute Terraform plan and save the output to tfplan.
                        echo 'Terraform plan completed.' // Log the successful completion of the planning process.
                    }
                }
            }
        }

        // Stage to apply Terraform configuration
        stage('Terraform Apply') {
            when {
                branch 'main' // Condition to only run this stage when on the 'main' branch.
                expression { currentBuild.rawBuild.getCause(hudson.model.Cause$UserIdCause) != null } // Condition to check if the build was triggered by a user.
            }
            steps {
                script {
                    input message: 'Do you want to apply these changes?', ok: 'Yes' // Request manual approval before applying changes.
                    echo 'Applying Terraform configuration...' // Log the start of Terraform application process.
                    withCredentials([aws(credentialsId: 'AWS_CRED', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                        sh 'terraform apply tfplan' // Apply the previously planned changes using the tfplan file.
                    }
                    echo 'Terraform configuration applied.' // Log the successful completion of the application process.
                }
            }
        }
    }

    // Post actions that run after the stages have completed
    post {
        always {
            echo 'Cleaning up workspace and temporary files...' // Log the start of cleanup process.
            cleanWs() // Clean up the workspace to remove any files created during the run.
            echo 'Cleanup completed.' // Log the successful completion of cleanup process.
        }
        failure {
            mail to: 'team@example.com', subject: 'Terraform Pipeline Failure', body: 'There was a failure in the Terraform pipeline.' // Send an email notification if the pipeline fails.
        }
    }
}
