#!groovy

// def slackMsg(status) {
//   def msg = [
//     aborted: ':heavy_minus_sign: Aborted!',
//     success: ':heavy_check_mark: Success!',
//     failure: ':heavy_multiplication_x: Failed!'
//   ]

//   return "${msg[status]}\n\n" +
//     "*Job Name:* ${env.JOB_NAME}\n" +
//     "*Build URL:* ${env.BUILD_URL}\n" +
//     "*AWS Environment:* ${aws_env}\n" +
//     "*AWS Region:* ${region}"
// }

def getConfig(repo_name){
  config = readYaml file: "./pipelines/config.yaml"
  return config[repo_name]
}

def getPipelineRoutes(repo_name){
  config = getConfig(repo_name.toLowerCase())
  if(config != null){
    return config
  } 
}

def isPullRequest() {
  result = false
  if ( "${payload_pull_request_state}" == "open" ) {
    echo "payload.pull_request.head.ref is BRANCH"
    if ( "${payload_pull_request_head_ref}" != "master" || "${payload_pull_request_head_ref}" != "main" ) {
      result = true
      return result
    } else {
      return result
    }
  } else {
    return result
  }
}


pipeline {
  agent any
  options {
    timeout(time: 30, unit: 'MINUTES')
  }
  stages {

    stage("Get Application\'s AWS Environment Details") {
      steps {
        script {
          def repoConfig = getPipelineRoutes("${payload_pull_request_head_repo_name}")
          def ASSUME_IAM_ROLE = repoConfig['ASSUME_IAM_ROLE']
          def AWS_ACCOUNT_NAME = repoConfig['AWS_ACCOUNT_NAME']
          def AWS_ACCOUNT_NUMBER = repoConfig['AWS_ACCOUNT_NUMBER']
          def AWS_REGION = repoConfig['AWS_REGION']
          def GITHUB_CREDENTIALS = repoConfig['GITHUB_CREDENTIALS']
          def CFN_CREDENTIALS_ID = "aws-id"
          echo "GET AWS ACCOUNT NAME AND NUMBER ...."
          echo "AWS ACCOUNT NAME: ${AWS_ACCOUNT_NAME}, AWS ACCOUNT NUMBER: ${AWS_ACCOUNT_NUMBER}"
          echo "GET AWS REGION ...."
          echo "AWS REGION: ${AWS_REGION}"
          echo "AWS ROLE: ${ASSUME_IAM_ROLE}"
        }
      }
    }

  // post {
  //   aborted {
  //     slackSend(channel: env.SLACK_CHANNEL, color: '#808080', message: slackMsg('aborted'), tokenCredentialId: env.SLACK_TOKEN_ID)
  //   }
  //   failure {
  //     slackSend(channel: env.SLACK_CHANNEL, color: 'danger', message: slackMsg('failure'), tokenCredentialId: env.SLACK_TOKEN_ID)
  //   }
  //   success {
  //     slackSend(channel: env.SLACK_CHANNEL, color: 'good', message: slackMsg('success'), tokenCredentialId: env.SLACK_TOKEN_ID)
  //   }
  // }
  }
}
