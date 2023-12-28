pipeline {
    agent any

    environment {
        BuildName = "version-${BUILD_NUMBER}"
        BucketName = "yashbucketdhhffh"
        BucketKey = "S3-builds-Storage"
        ApplicationName = "yash-java-application"
        EnvironmentName = "Yash-java-application-env"
        SourceDirectory = "java-se-jetty-gradle-v3"
    }

    stages {
        stage('Prepare') {
            steps {
                script {
                    // Change to the source directory
                    dir(SourceDirectory) {
                        // Zip the contents of the source directory
                        sh "zip -r ${BuildName}.zip *"
                        // Upload the zip file to S3
                        sh "aws s3 cp ${BuildName}.zip s3://${BucketName}/${BucketKey}/ --region us-east-1"
                    }
                }
            }
        }

        stage('Clean Up') {
            steps {
                script {
                    // Remove the local zip file
                    sh "rm ${WORKSPACE}/${SourceDirectory}/${BuildName}.zip"
                }
            }
        }

        stage('Create Application Version') {
            steps {
                script {
                    // Create Beanstalk Application Version
                    sh "aws elasticbeanstalk create-application-version --application-name '${ApplicationName}' --version-label '${BuildName}' --description 'Build created from JENKINS. Job:$JOB_NAME, BuildId:$BUILD_DISPLAY_NAME, GitCommit:$GIT_COMMIT, GitBranch:$GIT_BRANCH' --source-bundle S3Bucket=${BucketName},S3Key=${BucketKey}/${BuildName}.zip --region us-east-1"
                }
            }
        }

        stage('Update Environment') {
            steps {
                script {
                    // Update Beanstalk Environment
                    sh "aws elasticbeanstalk update-environment --environment-name '${EnvironmentName}' --version-label '${BuildName}' --region us-east-1"
                }
            }
        }
    }
}
