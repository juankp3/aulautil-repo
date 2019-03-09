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
          sh "echo ${env.WORKSPACE}"
          sh "touch archivo.txt"
          sh "mkdir -p micarpeta"
          sh "touch micarpeta/mifile.txt"
          sh "python main.py"
          script {
            def ID = sh(returnStdout: true, script: "./ami_id.sh ${env.BUILD_NUMBER}").trim() 
            sh "./build_ami.sh ${ID} prueba"
          }
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
