@Library('fxtest@1.6') _

pipeline {
  agent any
  options {
    ansiColor('xterm')
    timestamps()
    timeout(time: 5, unit: 'MINUTES')
  }
  environment {
    PYTEST_ADDOPTS = "-n=10 --color=yes"
    PULSE = credentials('PULSE')
  }
  stages {
    stage('Lint') {
      steps {
        sh "tox -e flake8"
      }
    }
    stage('Test') {
      steps {
        sh "tox -e py27"
      }
      post {
        always {
          archiveArtifacts 'results/*'
          junit 'results/*.xml'
          submitToActiveData('results/py27_raw.txt')
          submitToTreeherder('snippets-tests', 'e2e', 'End-to-end integration tests', 'results/*', 'results/py27_tbpl.txt')
        }
      }
    }
  }
  post {
    failure {
      mail(
        body: "${BUILD_URL}",
        from: "firefox-test-engineering@mozilla.com",
        replyTo: "firefox-test-engineering@mozilla.com",
        subject: "Build failed in Jenkins: ${JOB_NAME} #${BUILD_NUMBER}",
        to: "fte-ci@mozilla.com")
    }
    changed {
      ircNotification('#snippets')
      ircNotification('#fx-test-alerts')
    }
  }
}
