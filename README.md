# Jenkins Pipeline for CI/CD with Slack Notifications
This Jenkins pipeline automates the build and deployment process for a project, while sending notifications to a Slack channel during various stages of the pipeline. The pipeline performs the following actions:

* Clones the repository.
* Installs dependencies.
Builds the project.
Deploys the build artifacts to an S3 bucket.
Sends notifications to Slack indicating the start, success, or failure of the pipeline, along with the user who initiated the build and the stage where a failure occurred (if applicable).
Pipeline Overview
Stages:
Initialise: Sends a Slack notification that the build has been initiated by the user who triggered the pipeline.
Clone: Clones the project repository from the specified branch.
Build: Installs dependencies and builds the project.
Deploy: Uploads the build artifacts to an AWS S3 bucket.
Post Actions:
Sends a Slack notification on build success or failure. If the build fails, the notification will include the stage where the failure occurred.
Slack Notifications
Slack notifications are sent at the following points:

When the pipeline is initiated.
When the pipeline succeeds or fails.
On failure, the notification includes the stage where the error occurred.
Prerequisites
Jenkins Setup: Make sure Jenkins is set up with the following plugins:

Slack Notification Plugin: Configure it with your Slack workspace and channel.
Git Parameter Plugin: Allows branch selection via a parameter.
Git Plugin: To clone repositories.
AWS CLI: For uploading build artifacts to S3.
Jenkins Credentials:

credential_id: The credentials to access the repository.
An AWS IAM user with appropriate permissions to upload to S3.
Environment Variables:

BUILD_USER: Captures the Jenkins user who triggered the pipeline.
currentStage: Tracks the current stage in the pipeline for better failure handling.
Git Branch Parameter: The pipeline uses the gitParameter plugin to allow users to select the branch they want to build from.

Pipeline Script
groovy
Copy code
import groovy.json.JsonOutput

def COLOR_MAP = [
    'SUCCESS': 'good',
    'ALERT': 'warning',
    'FAILURE': 'danger'
]

def getBuildUser(){
    def userCause = currentBuild.rawBuild.getCause(Cause.UserIdCause)
    return userCause ? userCause.getUserId() : "Automated/Unknown"
}

pipeline {
    agent any
    
    environment {
        BUILD_USER = ''
        currentStage = '' // Variable to track current stage
    }

    parameters {
        gitParameter branchFilter: 'origin/(.*)', defaultValue: 'default_branch_name', name: 'BRANCH', type: 'PT_BRANCH'
    }

    stages {
        
        stage('Initialise') {
            steps {
                script{
                  BUILD_USER = getBuildUser()
                  currentStage = 'Initialise'
                }
                slackSend channel: 'cicd-test',
                        color: 'warning',
                      message: "${JOB_NAME} Build Initiated by ${BUILD_USER}"
            }
        }
        
        stage('clone') {
            steps {
                script {
                    currentStage = 'Clone'
                }
                git branch: "${params.BRANCH}", credentialsId: 'credential_id', url: 'repository_ssh_url'
            }
        }

        stage ('Build') {
            steps{
                script{
                  currentStage = 'Build'
                }
                sh 'npm i --legacy-peer-deps'
                sh 'npm run build:dev'
            }
        }

        stage('Deploy') {
            steps {
                script {
                    currentStage = 'Deploy'
                }
                sh 'aws s3 cp /var/lib/jenkins/workspace/$JOB_NAME/build/ s3://bucket_name/ --recursive'
            }
        }
    }

    post {
        success {
            script {
                BUILD_USER = getBuildUser()
            }
            slackSend channel: 'cicd-test',
                      color: COLOR_MAP['SUCCESS'],
                      message: "SUCCESS: Build ${JOB_NAME} ${env.BUILD_NUMBER} succeeded by ${BUILD_USER}"
        }
        failure {
            script {
                BUILD_USER = getBuildUser()
            }
            slackSend channel: 'cicd-test',
                      color: COLOR_MAP['FAILURE'],
                      message: "FAILURE: Build ${JOB_NAME} ${env.BUILD_NUMBER} failed in *${currentStage}* stage by ${BUILD_USER}"
        }
    }
}
Pipeline Configuration
Clone Repository: The pipeline will clone the branch selected via the BRANCH parameter from the provided repository.

Build: The npm commands used here (npm i --legacy-peer-deps and npm run build:dev) assume that the project uses Node.js. You can modify this to suit your specific project needs.

Deploy: The build artifacts are uploaded to an S3 bucket. Make sure you update the S3 bucket name (s3://bucket_name/) in the script to your target bucket.

Slack Notifications:

Ensure that the Slack channel cicd-test is configured and the Jenkins Slack plugin is properly set up with a valid token for your workspace.
Update the Slack channel name if necessary.
Customization
You can customize the pipeline for your project by making the following changes:

Repository Details: Update the git block with your repository URL and credentials.
Build Commands: Modify the build commands in the Build stage to fit your project's build process.
S3 Bucket: Update the S3 bucket name in the Deploy stage to your own S3 bucket.
Troubleshooting
Build User Not Captured: Ensure that Jenkins is configured to run with users properly authenticated to capture the BUILD_USER. If the build is triggered automatically (e.g., via webhook), the BUILD_USER will show as Automated/Unknown.
Slack Not Sending Messages: Ensure that the Slack plugin is correctly configured in Jenkins, and that the correct channel and credentials are used.
License
This project is licensed under the MIT License - see the LICENSE file for details.
