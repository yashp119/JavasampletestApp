pipeline {
    agent any

    environment {
        BuildName = "version-${BUILD_NUMBER}"
        BucketName = "yashbucketdhhffh"
        BucketKey = "S3-builds-Storage"
        ApplicationName = "practice"
        EnvironmentName = "Practice-env"
    }

    stages {
        stage('Checkout') {
            steps {
                // Assuming you are using Git for version control
                checkout scm
            }
        }

        stage('Build and Upload to S3') {
            steps {
                script {
                    // Assuming your project is in a subdirectory called 'java-se-jetty-gradle-v3'
                    dir('java-se-jetty-gradle-v3') {
                        // Use '*' to include all files (including hidden) in the current directory
                        sh "zip -r ${BuildName}.zip *"
                        sh "aws s3 cp ${BuildName}.zip s3://${BucketName} --region us-east-1"
                        sh "rm ${BuildName}.zip"
                    }
                }
            }
        }

        stage('Create Beanstalk Application Version') {
            steps {
                script {
                    sh "aws elasticbeanstalk create-application-version --application-name '${ApplicationName}' --version-label '${BuildName}' --description 'Build created from JENKINS. Job:${JOB_NAME}, BuildId:${BUILD_DISPLAY_NAME}, GitCommit:${GIT_COMMIT}, GitBranch:${GIT_BRANCH}' --source-bundle S3Bucket=${BucketName},S3Key=${BuildName}.zip --region us-east-1"
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
    }
}
