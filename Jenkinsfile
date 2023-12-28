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
                    def S3BucketPath = "" // Empty string for the root of the bucket
                    sh "aws s3 sync . s3://${BucketName}/${S3BucketPath} --region us-east-1"
                }
            }
        }

        stage('Create Beanstalk Application Version') {
            steps {
                script {
                    sh "aws elasticbeanstalk create-application-version --application-name '${ApplicationName}' --version-label '${BuildName}' --description 'Build created from JENKINS. Job:${JOB_NAME}, BuildId:${BUILD_DISPLAY_NAME}, GitCommit:${GIT_COMMIT}, GitBranch:${GIT_BRANCH}' --region us-east-1"
                }
            }
        }

        stage('Update Beanstalk Environment') {
            steps {
                script {
                    sh "aws elasticbeanstalk update-environment --environment-name '${EnvironmentName}' --version-label '${BuildName}' --region us-east-1"
                }
            }
        }

        stage('Cleanup Old Versions') {
            steps {
                script {
                    def versions = sh(script: "aws elasticbeanstalk describe-application-versions --application-name ${ApplicationName} --query 'ApplicationVersions[*].VersionLabel' --output text", returnStdout: true).trim().split()
                    versions.sort { a, b -> b <=> a }
                    def versionsToKeep = versions.take(2)
                    versions.each { version ->
                        if (!versionsToKeep.contains(version)) {
                            echo "Deleting old version: ${version}"
                            sh "aws elasticbeanstalk delete-application-version --application-name ${ApplicationName} --version-label ${version} --delete-source-bundle"
                        }
                    }
                }
            }
        }
    }
}
