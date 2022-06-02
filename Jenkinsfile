#!groovy

// def deploymentEnvs() {
//   return [
//     'TEST': [
//       env: 'nonprod:XXXXXXXXXX',
//       region: 'us-east-1',
//       iam_role: 'jenkins-build-role'
//     ]
//   ]
// }

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


def DEPLOYMENT_ENV = "TEST"
def CFN_CREDENTIALS_ID = "aws-id"
def jsonObj = readJSON text: payload

pipeline {
  agent any
  options {
    timeout(time: 30, unit: 'MINUTES')
    timestamps()
  }
  stages {

    stage('check version') {
      steps {
        sh '/usr/local/bin/aws --version'
        sh 'echo $payload'
        sh 'echo $jsonObj'
        sh 'echo ${jsonObj.payload.pull_request.head.repo.owner.url}'
      }
    }


    // stage("Get Application\'s AWS Environment Details") {
    //   steps {
    //     script {
    //       echo "GET AWS ACCOUNT NAME AND NUMBER ...."
    //       def deployment_env = deploymentEnvs()["${DEPLOYMENT_ENV}"]['env']
    //       (aws_env, account_number) = deployment_env.tokenize(':')
    //       echo "AWS ACCOUNT NAME: ${aws_env}, AWS ACCOUNT NUMBER: ${account_number}"

    //       echo "GET AWS REGION ...."
    //       region = deploymentEnvs()["${DEPLOYMENT_ENV}"]['region']
    //       echo "AWS REGION: ${region}"
    //       echo "AWS ROLE: ${ASSUME_IAM_ROLE}"
    //     }
    //   }
    // }

    // stage('update-stack') {
    //   steps {
    //     withAWS(role: "${ASSUME_IAM_ROLE}", region: "${REGION}", roleAccount: "${account_number}") {
    //       script {
    //         FILESET = sh(returnStdout: true, script: "git reflog --name-status -10 cloudformation|grep yaml |awk '{print \$2}' |sort |uniq").trim()
    //         echo "MODIFIED FILE FROM THE LAST COMMIT...."
    //         echo "${FILESET}"
    //       }
    //       script {
    //         if ( FILESET.isEmpty() ) {
    //           echo "No changes found on cfn templates.."
    //         } else {
    //           FILESET.each { cfn ->
    //             stage(cfn) {
    //               FILE_NAME = (cfn.split("\\/", 2)[1])
    //               STACK_NAME = (FILE_NAME.split("\\.", 2)[0])
    //               echo "STACK_NAME: ${STACK_NAME}"
    //               def stack_status = sh( returnStatus: true, script: "/usr/local/bin/aws cloudformation describe-stacks --region us-east-1 --stack-name ${STACK_NAME}")
    //               if ( stack_status == 0 ) {
    //                 sh "/usr/local/bin/aws cloudformation deploy --stack-name ${STACK_NAME} --region us-east-1 --template-file ./${cfn} --capabilities CAPABILITY_NAMED_IAM --no-fail-on-empty-changeset"
    //               } else {
    //                 echo "Stack ${STACK_NAME} don\'t exist.. The jenkins jobs support only update for now. If you add new cfn, create using console for the first time.."
    //               }
    //             }
    //           }
    //         }
    //       }
    //     }
    //   }
    // }

    stage('update-stack') {
      steps {
        withCredentials([[
        $class: 'AmazonWebServicesCredentialsBinding',
        credentialsId: "${CFN_CREDENTIALS_ID}",
        accessKeyVariable: 'AWS_ACCESS_KEY_ID',
        secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
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
                  def stack_status = sh( returnStatus: true, script: "/usr/local/bin/aws cloudformation describe-stacks --region us-east-1 --stack-name ${STACK_NAME}")
                  if ( stack_status == 0 ) {
                    sh "/usr/local/bin/aws cloudformation deploy --stack-name ${STACK_NAME} --region us-east-1 --template-file ./${cfn} --capabilities CAPABILITY_NAMED_IAM --no-fail-on-empty-changeset"
                  } else {
                    echo "Stack ${STACK_NAME} don\'t exist.. The jenkins jobs support only update for now. If you add new cfn create manully for the first time.."
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
