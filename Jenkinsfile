pipeline {
    agent any

    environment {
        AWS_ACCESS_KEY_ID     = credentials('aws-access-key')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-key')
        GITHUB_PAT            = credentials('github-pat')
        AWS_DEFAULT_REGION    = 'us-east-1'
    }

    stages {
        stage('Clean Workspace') {
            steps {
                echo 'Cleaning workspace...'
                deleteDir()
            }
        }

        stage('Setup Python Virtual Environment') {
            steps {
                echo 'Setting up Python virtual environment...'
                sh '''
                python3 -m venv venv
                . venv/bin/activate
                pip install --upgrade pip boto3
                '''
            }
        }

        stage('Checkout Code') {
            steps {
                dir('terraform') {
                    echo 'Checking out code from GitHub...'
                    git(
                        url: 'https://github.com/AnandJoy7/V1.git',
                        branch: 'main',
                        credentialsId: 'github-credentials'
                    )
                }
            }
        }

        stage('Run Terraform Script') {
            steps {
                dir('terraform') {
                    script {
                        echo 'Running Terraform script...'
                        def result = sh(script: '''
                        . ../venv/bin/activate
                        python3 terra_auto8.py > terraform_output.log 2>&1
                        deactivate
                        ''', returnStatus: true)
                        
                        if (result != 0) {
                            error "Terraform script failed. Check terraform_output.log for details."
                        } else {
                            echo "Terraform script ran successfully."
                        }
                    }
                }
            }
        }

        stage('Set Up Git and Push Selected Files to New Repository') {
            steps {
                dir('terraform') {
                    script {
                        echo 'Configuring Git and setting up the remote...'
                        sh '''
                            git config user.name "AnandJoy7"
                            git config user.email "k.anad548@gmail.com"
                            git remote remove origin || true
                            git remote add origin https://${GITHUB_PAT}@github.com/AnandJoy7/terra_auto_final_1.git
                            
                            # Add only specified files to the commit
                            git add Child_Module/main.tf
                            git add Child_Module/terraform.tfstate
                            git add Child_Module/terraform.tfstate.backup
                            git add Child_Module/terraform.tfvars
                            git add Child_Module/variables.tf
                            git add Parent_Module/main.tf
                            git add Parent_Module/variables.tf
                            git add terraform_output.log
                            
                            # Commit and push the selected files
                            git commit -m "Added selected Terraform files and output log" || echo "Nothing to commit"
                            git push -u origin main --force
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Archiving Terraform output log...'
            dir('terraform') {
                archiveArtifacts artifacts: 'terraform_output.log', allowEmptyArchive: true
            }
        }
        success {
            echo 'Terraform automation completed successfully and selected files pushed to new repository.'
        }
        failure {
            echo 'Terraform automation failed. Please check the logs.'
        }
    }
}
