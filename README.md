Jenkins Pipeline for CI/CD with Slack Notifications
===================================================

[](https://github.com/nasibsingh/jenkins-frontend-deployment/edit/main/README.md#jenkins-pipeline-for-cicd-with-slack-notifications)

This Jenkins pipeline automates the build and deployment process for a project, while sending notifications to a Slack channel during various stages of the pipeline. The pipeline performs the following actions:

-   Clones the repository.
-   Installs dependencies.
-   Builds the project.
-   Deploys the build artifacts to an S3 bucket.
-   Sends notifications to Slack indicating the start, success, or failure of the pipeline, along with the user who initiated the build and the stage where a failure occurred (if applicable).

Pipeline Overview
-----------------

[](https://github.com/nasibsingh/jenkins-frontend-deployment/edit/main/README.md#pipeline-overview)

### Stages:

[](https://github.com/nasibsingh/jenkins-frontend-deployment/edit/main/README.md#stages)

1.   **Initialise**: Sends a Slack notification that the build has been initiated by the user who triggered the pipeline.
2.   **Clone**: Clones the project repository from the specified branch.
3.   **Build**: Installs dependencies and builds the project.
4.   **Deploy**: Uploads the build artifacts to an AWS S3 bucket.

### Post Actions:

[](https://github.com/nasibsingh/jenkins-frontend-deployment/edit/main/README.md#post-actions)

-   Sends a Slack notification on build success or failure. If the build fails, the notification will include the stage where the failure occurred.

Slack Notifications
-------------------

[](https://github.com/nasibsingh/jenkins-frontend-deployment/edit/main/README.md#slack-notifications)

Slack notifications are sent at the following points:

-   When the pipeline is initiated.
-   When the pipeline succeeds or fails.
-   On failure, the notification includes the stage where the error occurred.

Prerequisites
-------------

[](https://github.com/nasibsingh/jenkins-frontend-deployment/edit/main/README.md#prerequisites)

1.   **Jenkins Setup**: Make sure Jenkins is set up with the following plugins: 
     - **Slack Notification Plugin**: Configure it with your Slack workspace and channel.
     - **Git Parameter Plugin**: Allows branch selection via a parameter.
     - **Git Plugin**: To clone repositories.
     - **AWS CLI**: For uploading build artifacts to S3.

2.   Jenkins Credentials:
     - `credential_id`: The credentials to access the repository.
     - An AWS IAM user with appropriate permissions to upload to S3.

3.   Environment Variables:
     - `BUILD_USER`: Captures the Jenkins user who triggered the pipeline.
     - `currentStage`: Tracks the current stage in the pipeline for better failure handling.
4.   Git Branch Parameter: The pipeline uses the gitParameter plugin to allow users to select the branch they want to build from.

Pipeline Configuration
----------------------

[](https://github.com/nasibsingh/jenkins-frontend-deployment/edit/main/README.md#pipeline-configuration)

1.   Clone Repository: The pipeline will clone the branch selected via the `BRANCH` parameter from the provided repository.

2.   Build: The `npm` commands used here (`npm i --legacy-peer-deps` and `npm run build:dev`) assume that the project uses Node.js. You can modify this to suit your specific project needs.

3.   Deploy: The build artifacts are uploaded to an S3 bucket. Make sure you update the S3 bucket name (`s3://bucket_name/`) in the script to your target bucket.

4.   Slack Notifications:
     - Ensure that the Slack channel `channel_name` is configured and the Jenkins Slack plugin is properly set up with a valid token for your workspace.
     - Update the Slack channel name if necessary.

Customization
-------------

[](https://github.com/nasibsingh/jenkins-frontend-deployment/edit/main/README.md#customization)

You can customize the pipeline for your project by making the following changes:

-   Repository Details: Update the `git` block with your repository URL and credentials.
-   Build Commands: Modify the build commands in the `Build` stage to fit your project's build process.
-   S3 Bucket: Update the S3 bucket name in the `Deploy` stage to your own S3 bucket.

Troubleshooting
---------------

[](https://github.com/nasibsingh/jenkins-frontend-deployment/edit/main/README.md#troubleshooting)

-   Build User Not Captured: Ensure that Jenkins is configured to run with users properly authenticated to capture the `BUILD_USER`. If the build is triggered automatically (e.g., via webhook), the `BUILD_USER` will show as `Automated/Unknown`.
-   Slack Not Sending Messages: Ensure that the Slack plugin is correctly configured in Jenkins, and that the correct channel and credentials are used.

## License This project is licensed under the MIT License - see the LICENSE file for details.
