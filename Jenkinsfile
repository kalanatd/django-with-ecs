pipeline {
    agent {label "linux"}
    parameters {
        string(name: 'ECSClusterName', defaultValue: 'django-web-app')
        string(name: 'SlackChannel', defaultValue: 'Jenkins Job Alerts', description: 'Slack Channel to receive the job summary')
        string(name: 'KeyName', defaultValue: 'webapp-key')
        string(name: 'VpcId', defaultValue: 'vpc-ef12334', description: 'Select a VPC that allows instances to access the Internet.')
        string(name: 'SubnetId', defaultValue: 'subnet-14b8as7a4dsd, subnet-0c38sf4e3bb8d419db3' ,description: 'Enter at least two comma serparated subnets in your selected VPC.')
        choice(name: 'DesiredCapacity', choices: [1,2,3,4,5], description: 'Desired Capacity of the ECS Cluster Target Group')
        choice(name: 'InstanceType', choices: ['t2.micro', 't2.small', 't2.medium','t2.large', 'm3.medium.', 'm3.xlarge', 'm4.large'], description: 'EC2 Instance type')
        choice(name: 'MaxSize', choices: [1,2,3,4,5], description: 'Maximum number of instances that can be launched in your ECS cluster.')
    }
    stages {
        stage('Docker Build') {
            // Building the Docker Image and Pushing to the kalanatd/django Dockerhub repo.
            agent {
                label "docker && linux"
            }
            steps {
                sh '''
                    docker build . -t kalanatd/django -f django-app/Dockerfile
                    docker push kalanatd/django
                '''
            }
        }
        stage('Run Cloudformation Script') {
            steps {
                // Running the Cloudformation Script
                script {    
                    // Get aws cli authentication creds using a Jenkins Variable
                    for (int i = 0; i < AWS_CREDS.size(); i++) {
                        env.AWS_CRED = "${AWS_CREDS[i]}"
                        withAWSCredentials("${AWS_CREDS[i]}") {
                            env.exit_status = sh(script: "aws cloudformation create-stack --template-body file://cloudformation-scripts/base.yaml --stack-name ${ECSClusterName} --parameters ParameterKey=KeyName,ParameterValue=${KeyName} ParameterKey=VpcId,ParameterValue=${VpcId} ParameterKey=SubnetId,ParameterValue=${SubnetId} ParameterKey=DesiredCapacity,ParameterValue=${DesiredCapacity} ParameterKey=InstanceType,ParameterValue=${InstanceType} ParameterKey=MaxSize,ParameterValue=${MaxSize}" ).trim()
                        }
                    }

                }
            }
        }
        stage ('Sending Slack Alerts Custom Image') {
            steps {
                sh 'echo "Sending Slack alerts"'
                script {
                    env.noOfVulnerabilities = sh(label: 'Vulner',script: "cut -d',' -f13,14 scan_report.csv | grep 'high\\|critical' | grep -v OS | wc -l", returnStdout: true).trim()
                    if (noOfVulnerabilities) {
                        sendSlackAlertCustom('danger', ":alert: DAI Nightly Prisma Cloud Security Scan Results")
                    }
                    if (false) {
                        sh 'echo "No vulnerabilities"'
                    }
                }
            }
        }
    }
    post {
        success {
            echo 'Job is Successful'
            sendSlackAlert('Success', "::white_check_mark: Job is Successful")
        }
        failure {
            echo 'Job is Failed'
            sendSlackAlert('danger', ":alert: Job is Failed")
        }
    }
}
def sendSlackAlert(statusColor,HeaderMessage){
    withCredentials([string(credentialsId: 'SLACK-BOT-TOKEN', variable: 'slackCredentials')]) {
        def channel = "${Slack_Channel}"
        def body = "*Build Status* ${currentBuild.currentResult} \n *Job Name: * ${env.JOB_NAME} \n *Build Number: * ${env.BUILD_NUMBER}" 
        def buildInfo = [[text: body, color: statusColor, fallback: 'Build Alert']]
        def slackResponse = slackSend(channel: channel, message: "*" + HeaderMessage + "*", attachments: buildInfo, botUser: true, tokenCredentialId: "SLACK-BOT-TOKEN")
    }
}
