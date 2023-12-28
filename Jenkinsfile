pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = 'us-east-1'
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    checkout scm
                }
            }
        }

        stage('Upload to S3') {
            steps {
                script {
                    sh 'aws s3 sync . s3://yashbucketdhhffh/ --region $AWS_DEFAULT_REGION'
                }
            }
        }

        stage('Create Beanstalk Application Version') {
            steps {
                script {
                    def versionLabel = "version-${BUILD_NUMBER}"

                    sh """
                        aws elasticbeanstalk create-application-version
                        --application-name yash-java-application
                        --version-label $versionLabel
                        --description 'Build created from JENKINS. Job:${JOB_NAME}, BuildId:#${BUILD_NUMBER}, GitCommit:${GIT_COMMIT}, GitBranch:${GIT_BRANCH}'
                        --region $AWS_DEFAULT_REGION
                    """
                }
            }
        }

        stage('Update Beanstalk Environment') {
            steps {
                script {
                    def versionLabel = "version-${BUILD_NUMBER}"

                    sh """
                        aws elasticbeanstalk update-environment
                        --environment-name Yash-java-application-env
                        --version-label $versionLabel
                        --region $AWS_DEFAULT_REGION
                    """
                }
            }
        }

        stage('Cleanup Old Versions') {
            steps {
                script {
                    def versions = sh(script: "aws elasticbeanstalk describe-application-versions --application-name yash-java-application --query 'ApplicationVersions[*].VersionLabel' --output text", returnStdout: true).trim().split()
                    versions.remove("version-${BUILD_NUMBER}")  // Exclude the current version

                    for (def oldVersion in versions) {
                        echo "Deleting old version: $oldVersion"
                        sh "aws elasticbeanstalk delete-application-version --application-name yash-java-application --version-label $oldVersion --region $AWS_DEFAULT_REGION"
                    }
                }
            }
        }
    }
}
