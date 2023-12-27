pipeline {
    agent any  // Declare the agent to run the pipeline

    environment {
        // Define environment variables for sensitive information
        BUILD_NAME = "version-$BUILD_NUMBER"
        BUCKET_NAME = "yashbucketdhhffh"
        BUCKET_KEY = "S3-builds-Storage"
        APPLICATION_NAME = "practice"
        ENVIRONMENT_NAME = "Practice-env"
    }

    stages {
        stage('Build') {
            steps {
                echo "Building the application..."
                // Assuming build steps are within a Gradle project
                sh 'cd java-se-jetty-gradle-v3 && gradle build'
            }
        }

        stage('Package') {
            steps {
                echo "Packaging the application..."
                sh "cd java-se-jetty-gradle-v3 && zip -r $BUILD_NAME.zip *"
            }
        }

        stage('Upload to S3') {
            steps {
                echo "Uploading the package to S3..."
                withAWS(credentials: 'your-aws-credentials', region: 'ap-south-1') {
                    s3Upload(bucket: env.BUCKET_NAME, file: "java-se-jetty-gradle-v3/$BUILD_NAME.zip", path: env.BUCKET_KEY)
                }
            }
        }

        stage('Deploy to Elastic Beanstalk') {
            steps {
                echo "Deploying to Elastic Beanstalk..."
                withAWS(credentials: 'your-aws-credentials', region: 'ap-south-1') {
                    ebCreateApplicationVersion(applicationName: env.APPLICATION_NAME, versionLabel: env.BUILD_NAME, description: "Build created from JENKINS. Job:$JOB_NAME, BuildId:$BUILD_DISPLAY_NAME, GitCommit:$GIT_COMMIT, GitBranch:$GIT_BRANCH", sourceBundle: env.BUCKET_NAME + "/" + env.BUCKET_KEY + "/" + env.BUILD_NAME + ".zip")
                    ebUpdateEnvironment(environmentName: env.ENVIRONMENT_NAME, versionLabel: env.BUILD_NAME)
                }
            }
        }

        stage('Cleanup') {
            steps {
                echo "Cleaning up workspace..."
                sh "rm /var/lib/jenkins/workspace/new-file-jenkins/java-se-jetty-gradle-v3/$BUILD_NAME.zip"
            }
        }
    }
}
