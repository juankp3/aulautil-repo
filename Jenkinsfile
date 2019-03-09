pipeline {
  agent any
    options {
      ansiColor('xterm')
      timestamps()
      timeout(time: 1, unit: 'HOURS')
    }
    environment {
      ARTIFACT = "${env.BUILD_NUMBER}.zip"
      SLACK_MESSAGE = "Job '${env.JOB_NAME}' Build ${env.BUILD_NUMBER} URL ${env.JENKINS_URL}"
    }
    parameters {
      string(name: 'SLACK_CHANNEL', defaultValue:'#deploys', description: '')
      choice(name: 'TYPE', choices: 'aut\ncron\ndata',description: 'Autoscaling, Cron or Data')
      booleanParam(name: 'LAUNCH_CONFIGURATION', defaultValue: false, description: 'Update aws launchconfiguration with the new ami')
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
            syntax: { sh "./scripts/check_syntax.sh" },
            grep: { sh "echo grep_var_dump" }
          )
        }
      }
      stage('Build') {
        steps {
          sh "echo ${env.BUILD_NUMBER}"
          sh "echo ${env.WORKSPACE}"
          sh "touch archivo.txt"
          sh "mkdir -p micarpeta"
          sh "touch micarpeta/mifile.txt"
          sh "python main.py"
        }
      }
      stage('Deploy') {
        steps {
          sh "ls -la"
          sh "ls -la micarpeta"
          sh 'echo deploy'
          echo "${env.SLACK_MESSAGE}"
          echo "${params.SLACK_CHANNEL}"
          echo "${params.TYPE}"
          echo "${params.LAUNCH_CONFIGURATION}"
        }
      }
   }
}
