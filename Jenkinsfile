pipeline {
    agent any

    environment {
        BuildName = "version-${BUILD_NUMBER}"
        BucketName = "yashbucketdhhffh"
        ApplicationName = "yash-java-application"
        EnvironmentName = "Yash-java-application-env"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Upload to S3') {
            steps {
                script {
                    def S3BucketPath = ""
                    sh "aws s3 sync . s3://${BucketName}/${S3BucketPath} --region us-east-1"
                }
            }
        }

        stage('Create Beanstalk Application Version') {
            steps {
                script {
                    def response = sh(script: "aws elasticbeanstalk create-application-version --application-name '${ApplicationName}' --version-label '${BuildName}' --description 'Build created from JENKINS. Job:${JOB_NAME}, BuildId:${BUILD_DISPLAY_NAME}, GitCommit:${GIT_COMMIT}, GitBranch:${GIT_BRANCH}' --region us-east-1", returnStatus: true)
                    if (response != 0) {
                        error "Failed to create Beanstalk application version"
                    }
                }
            }
        }

        stage('Update Beanstalk Environment') {
            steps {
                script {
                    def response = sh(script: "aws elasticbeanstalk update-environment --environment-name '${EnvironmentName}' --version-label '${BuildName}' --region us-east-1", returnStatus: true)
                    if (response != 0) {
                        error "Failed to update Beanstalk environment"
                    }
                }
            }
        }

        stage('Delete Old Beanstalk Versions') {
            steps {
                script {
                    def versionsToDelete = sh(script: "aws elasticbeanstalk describe-application-versions --application-name ${ApplicationName} --query 'Versions[*].[VersionLabel]' --output text --region us-east-1", returnStdout: true).trim().split()

                    // Sort versions in ascending order
                    versionsToDelete = versionsToDelete.sort()

                    // Keep the latest two versions, delete the rest
                    versionsToDelete[0..(versionsToDelete.size() - 3)].each { version ->
                        sh "aws elasticbeanstalk delete-application-version --application-name ${ApplicationName} --version-label ${version} --region us-east-1"
                    }
                }
            }
        }
    }
}
