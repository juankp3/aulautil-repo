pipeline {
  agent any
    options {
      ansiColor('xterm')
      timestamps()
      timeout(time: 1, unit: 'HOURS')
    }
    environment {
      ARTIFACT = "${env.BUILD_NUMBER}.zip"
      SLACK_MESSAGE = "Job '${env.JOB_NAME}' Build ${env.BUILD_NUMBER} URL ${env.BUILD_URL}"
    }
    parameters {
      string(name: 'SLACK_CHANNEL', defaultValue:'#deploys', description: '')
      choice(name: 'TYPE', choices: 'aut\ncron\ndata',description: 'Autoscaling, Cron or Data')
      booleanParam(name: 'LC', defaultValue: false, description: 'Update aws launchconfiguration with the new ami')
    }
    stages {
      stage('Repository') {
        steps {
          checkout scm
        }
      }
      stage('Test') {
        steps {
          parallel (
            syntax: { sh "echo syntax" },
            grep: { sh "echo grep_var_dump" },
            valitaion: { sh "echo p_valitaion" },
            show_info: { sh "echo p_show_info" }
          )
        }
      }
      stage('Build') {
        steps {
          sh "echo ${env.BUILD_NUMBER}"
          sh "spmkdir -p prueba"
          sh "echo ${env.WORKSPACE}"
          sh "touch archivo.txt"
          sh "mkdir -p micarpeta"
          sh "touch micarpeta/mifile.txt"
          sh "python main.py"
          script {
            def ID = sh(returnStdout: true, script: "./ami_id.sh ${env.BUILD_NUMBER}").trim() 
            sh "./build_ami.sh ${ID} prueba"
          }
          echo "${env.SLACK_MESSAGE}"
          echo "${params.SLACK_CHANNEL}"
          echo "${params.TYPE}"
          echo "${params.LC}"
        }
      }
      stage('Deploy') {
        when {
          expression {
            return params.LC ==~ /(?i)(Y|YES|T|TRUE|ON|RUN)/
          }
        }
        steps {
          sh "echo hagamos_deploy"
          build job: "pipeline-prod", parameters: [
            [$class: 'StringParameterValue', name: 'VPC_ID', value: 'vpc-123'],
            [$class: 'StringParameterValue', name: 'SLACK', value: '#deploys']
          ]
        }
      }
    }

    post {
      always {
          sh "zip -r ${env.ARTIFACT} ./"
          archiveArtifacts artifacts: "${env.ARTIFACT}", onlyIfSuccessful: true
          sh "rm -f ${env.ARTIFACT}"
          echo "Job has finished"
      }
      success {
        slackSendMessage "good"
      }
      failure {
        slackSendMessage "danger"
      }
    }

}

def slackSendMessage(String color){
  slackSend channel: "${params.SLACK_CHANNEL}",
  color: color,
  failOnError: true,
  message: "$SLACK_MESSAGE"
}
