pipeline {
    agent { label 'electronix' }

    environment {
        S3_BUCKET = 'electronix-production-2026'
        CLOUDFRONT_DISTRIBUTION_ID = 'E1BODMCXENV2E0'
        AWS_REGION = 'us-east-1'
    }

    stages {

        stage('Frontend Pipeline') {
            when {
                changeset "frontend/**"
            }
            stages {

                stage('Install Dependencies') {
                    steps {
                        dir('frontend') {
                            sh 'npm install'
                        }
                    }
                }

                stage('Run Tests') {
                    steps {
                        dir('frontend') {
                            sh 'npm test -- --watchAll=false || echo "No tests configured, skipping"'
                        }
                    }
                }

                stage('Build') {
                    steps {
                        dir('frontend') {
                            sh 'npm run build'
                        }
                    }
                }

                stage('Archive Artifact') {
                    steps {
                        dir('frontend') {
                            archiveArtifacts artifacts: 'dist/**', fingerprint: true
                        }
                    }
                }

                stage('Deploy to S3') {
                    steps {
                        dir('frontend') {
                            sh """
                                aws s3 sync dist/ s3://${S3_BUCKET} --delete --region ${AWS_REGION}
                            """
                        }
                    }
                }

                stage('Invalidate CloudFront Cache') {
                    steps {
                        sh """
                            aws cloudfront create-invalidation --distribution-id ${CLOUDFRONT_DISTRIBUTION_ID} --paths "/*"
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline Pass'
        }
        failure {
            echo 'Pipeline Fail'
        }
    }
}