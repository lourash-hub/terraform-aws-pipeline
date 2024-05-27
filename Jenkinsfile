pipeline {
    agent any // Defines that this pipeline can run on any available agent.

    environment {
        TF_CLI_ARGS = '-no-color' // Set environment variable to disable color in Terraform CLI output for better readability in Jenkins logs.
    }

    stages {
        // Existing stages remain unchanged...

        // Stage to apply Terraform configuration
        stage('Terraform Apply') {
            when {
                branch 'main' // Condition to only run this stage when on the 'main' branch.
                expression { currentBuild.rawBuild.getCause(hudson.model.Cause$UserIdCause) != null } // Condition to check if the build was triggered by a user.
            }
            steps {
                script {
                    input message: 'Do you want to apply these changes?', ok: 'Yes'
                    echo 'Applying Terraform configuration...'
                    withCredentials([aws(credentialsId: 'AWS_CRED', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                        sh 'terraform apply tfplan'
                    }
                    echo 'Terraform configuration applied.'
                }
            }
        }

        // Stage to destroy Terraform-managed infrastructure
        stage('Terraform Destroy') {
            when {
                branch 'main' // Ensures this critical operation only runs in the main branch.
                expression { currentBuild.rawBuild.getCause(hudson.model.Cause$UserIdCause) != null } // Ensures it's a user-triggered build.
            }
            steps {
                script {
                    input message: 'Are you sure you want to destroy the Terraform-managed infrastructure?', ok: 'Yes'
                    echo 'Destroying Terraform-managed infrastructure...'
                    withCredentials([aws(credentialsId: 'AWS_CRED', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                        sh 'terraform destroy -auto-approve'
                    }
                    echo 'Terraform-managed infrastructure destroyed.'
                }
            }
        }
    }

    // Post actions that run after the stages have completed
    post {
        always {
            echo 'Cleaning up workspace and temporary files...'
            cleanWs() // Clean up the workspace to remove any files created during the run.
            echo 'Cleanup completed.'
        }
        failure {
            mail to: 'team@example.com', subject: 'Terraform Pipeline Failure', body: 'There was a failure in the Terraform pipeline.' // Send an email notification if the pipeline fails.
        }
    }
}
