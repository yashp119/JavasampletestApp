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
                // Checkout your source code from version control (e.g., Git)
                checkout scm
            }
        }

        stage('Build and Package') {
            steps {
                script {
                    // Change to your project directory
                    dir('java-se-jetty-gradle-v3') {
                        // Build and package your application
                        sh 'gradle build'

                        // Zip the artifacts
                        sh 'zip -r ${BuildName}.zip *'
                    }
                }
            }
        }

        stage('Upload to S3') {
            steps {
                script {
                    // Upload the artifact to S3
                    sh "aws s3 cp ${BuildName}.zip s3://${BucketName}/${BucketKey}/ --region ap-south-1"
                }
            }
        }

        stage('Deploy to Elastic Beanstalk') {
            steps {
                script {
                    // Create application version
                    sh "aws elasticbeanstalk create-application-version --application-name '${ApplicationName}' --version-label '${BuildName}' --description 'Build created from JENKINS. Job:${JOB_NAME}, BuildId:${BUILD_DISPLAY_NAME}, GitCommit:${GIT_COMMIT}, GitBranch:${GIT_BRANCH}' --source-bundle S3Bucket=${BucketName},S3Key=${BucketKey}/${BuildName}.zip --region ap-south-1"

                    // Update environment with new version
                    sh "aws elasticbeanstalk update-environment --environment-name '${EnvironmentName}' --version-label '${BuildName}' --region ap-south-1"
                }
            }
        }
    }
}
