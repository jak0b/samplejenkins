pipeline {
    agent any

    environment {
        AWS_ACCESS_KEY_ID = credentials('AWS_ACCESS_KEY_ID') // Set up AWS access key ID credentials in Jenkins
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY') // Set up AWS secret access key credentials in Jenkins
        AWS_DEFAULT_REGION = 'eu-west-1'
        AWS_S3_BUCKET = '1xfn5YXlIMIXSQSQhgBJd-cf-stacks'
        STACK_YAML_FILE = 'stack.yaml'
        STACK_NAME = "${env.BRANCH_NAME}" // Use the current branch name as the stack name
        ENVIRONMENT = "${env.BRANCH_NAME}"
        INSTANCE_TYPE = 't2.micro'
        INSTANCE_AMI = 'ami-0fb2f0b847d44d4f0'
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/jak0b/samplejenkins.git'
            }
        }

        stage('Copy to S3') {
            steps {
                script {
                    def latestStackYaml = findLatestStackYaml()
                    if (latestStackYaml != null) {
                        withAWS(credentials: 'your_aws_credentials_profile', region: 'your_aws_region') {
                            def s3Url = s3Upload(file: latestStackYaml, bucket: AWS_S3_BUCKET, path: STACK_YAML_FILE, acl: 'private').getUrl()
                            env.STACK_URL = s3Url // Store the S3 URL as an environment variable for use in the deployment stage
                        }
                    } else {
                        error("No stack.yaml found!")
                    }
                }
            }
        }

        stage('Deploy CloudFormation Stack') {
            when {
                expression { env.STACK_URL != null } // Run this stage only if the S3 URL is available
            }
            steps {
                script {
                    sh "aws cloudformation deploy --stack-name ${STACK_NAME} --template-url ${env.STACK_URL} --parameter-overrides Environment=${ENVIRONMENT} InstanceType=${INSTANCE_TYPE} InstanceAmi=${INSTANCE_AMI} --capabilities CAPABILITY_NAMED_IAM"
                }
            }
        }
    }
}

def findLatestStackYaml() {
    def files = findFiles(glob: '**/stack.yaml')
    if (files) {
        files.sort { it.lastModified() }
        return files.last().name
    } else {
        return null
    }
}
