pipeline {
    agent any

    environment {
        // S3 bucket the static site is deployed to
        S3_BUCKET  = 'jenkins-static-site-799183605681'
        AWS_REGION = 'ap-south-1'
        // Jenkins credentials IDs (configure these in: Manage Jenkins > Credentials)
        AWS_ACCESS_KEY_ID     = credentials('aws-access-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
    }

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify AWS access') {
            steps {
                sh 'aws sts get-caller-identity --region $AWS_REGION'
            }
        }

        stage('Deploy to S3') {
            steps {
                // Upload everything except VCS/CI metadata.
                // --delete removes files from the bucket that no longer exist in the repo.
                sh '''
                    aws s3 sync . "s3://$S3_BUCKET" \
                        --region "$AWS_REGION" \
                        --delete \
                        --exclude ".git/*" \
                        --exclude "Jenkinsfile" \
                        --exclude "README.md" \
                        --exclude ".gitignore"
                '''
            }
        }

        stage('Set cache headers') {
            steps {
                // HTML should not be cached aggressively so updates show immediately.
                sh '''
                    aws s3 cp "s3://$S3_BUCKET" "s3://$S3_BUCKET" \
                        --region "$AWS_REGION" \
                        --recursive \
                        --exclude "*" --include "*.html" \
                        --metadata-directive REPLACE \
                        --cache-control "no-cache" \
                        --content-type "text/html"
                '''
            }
        }
    }

    post {
        success {
            echo "Deployed to http://${S3_BUCKET}.s3-website.${AWS_REGION}.amazonaws.com"
        }
        failure {
            echo 'Deployment failed. Check the stage logs above.'
        }
    }
}
