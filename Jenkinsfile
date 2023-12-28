pipeline {
    agent any

    environment {
        BuildName = "version-${BUILD_NUMBER}"
        BucketName = "mymybucketgg"
        ApplicationName = "yash-java-application"
        EnvironmentName = "Practice-env"
        ZipFileName = "app.zip"
    }

    stages {
        stage('Checkout') {
            steps {
                // Assuming you are using Git for version control
                checkout scm
            }
        }

        stage('Create Zip File') {
            steps {
                script {
                    // Exclude Jenkinsfile from the zip file
                    sh "zip -r ${ZipFileName} . -x Jenkinsfile"
                }
            }
        }

        stage('Upload Zip to S3') {
            steps {
                script {
                    sh "aws s3 cp ${ZipFileName} s3://${BucketName}/${ZipFileName} --region us-east-1"
                }
            }
        }

        stage('Create Beanstalk Application Version') {
            steps {
                script {
                    sh "aws elasticbeanstalk create-application-version --application-name '${ApplicationName}' --version-label '${BuildName}' --source-bundle S3Bucket='${BucketName}',S3Key='${ZipFileName}' --region us-east-1"
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
