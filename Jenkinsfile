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

def repoConfig = getPipelineRoutes("${payload_pull_request_head_repo_name}")
def ASSUME_IAM_ROLE = repoConfig('ASSUME_IAM_ROLE')
def AWS_ACCOUNT_NAME = repoConfig('AWS_ACCOUNT_NAME')
def AWS_ACCOUNT_NUMBER = repoConfig('AWS_ACCOUNT_NUMBER')
def AWS_REGION = repoConfig('AWS_REGION')
def GITHUB_CREDENTIALS = repoConfig('GITHUB_CREDENTIALS')
def CFN_CREDENTIALS_ID = "aws-id"


pipeline {
  agent {label 'packer'}
  options {
    timeout(time: 30, unit: 'MINUTES')
  }
  stages {

    stage("Get Application\'s AWS Environment Details") {
      steps {
        script {
          echo "GET AWS ACCOUNT NAME AND NUMBER ...."
          echo "AWS ACCOUNT NAME: ${AWS_ACCOUNT_NAME}, AWS ACCOUNT NUMBER: ${AWS_ACCOUNT_NUMBER}"
          echo "GET AWS REGION ...."
          echo "AWS REGION: ${AWS_REGION}"
          echo "AWS ROLE: ${ASSUME_IAM_ROLE}"
        }
      }
    }

    stage('update-stack') {
      when {
          equals expected: 'true', actual: isPullRequest()
      }
      steps {
        container('aws-cli') {
          withCredentials([[
          $class: 'AmazonWebServicesCredentialsBinding',
            credentialsId: "${CFN_CREDENTIALS_ID}", // update the credentail_id
            accessKeyVariable: 'AWS_ACCESS_KEY_ID',
            secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
            git url: "${payload_pull_request_head_repo_html_url}", credentialsId: "${GITHUB_CREDENTIALS}"
            script {
              sh "git checkout ${payload_pull_request_head_ref}"
            }
            script {
              FILESET = sh(returnStdout: true, script: "git reflog --name-status -10 cloudformation|grep yaml |awk '{print \$2}' |sort |uniq").trim()
              echo "MODIFIED FILE FROM THE LAST COMMIT...."
              echo "${FILESET}"
            }
            script {
              if ( FILESET.isEmpty() ) {
                echo "No changes found on cfn templates.."
              } else {
                FILESET.each { cfn ->
                  stage(cfn) {
                    FILE_NAME = (cfn.split("\\/", 2)[1])
                    STACK_NAME = (FILE_NAME.split("\\.", 2)[0])
                    echo "STACK_NAME: ${STACK_NAME}"
                    def stack_status = sh( returnStatus: true, script: "/usr/local/bin/aws cloudformation describe-stacks --region  ${AWS_REGION} --stack-name ${STACK_NAME}")
                    if ( stack_status == 0 ) {
                      sh "/usr/local/bin/aws cloudformation validate-template --stack-name ${STACK_NAME} --region  ${AWS_REGION} --template-file ./${cfn}"
                    } else {
                      echo "Stack ${STACK_NAME} don\'t exist.. The Jenkins jobs support only update for now. If you add new cfn create manually for the first time.."
                    }
                  }
                }
              }
            }
          }
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
