pipeline {
  agent any

  tools {
    nodejs "node-lts"
  }

  environment {
    REPO = "https://github.com/ShivyaDahiya/8.2CDevSecOps.git"
    GIT_CRED = "github-cred"
    NOTIFY_EMAIL = "shivyadahiya2002@gmail.com"
  }

  stages {

    stage('Checkout') {
      steps {
        // Checkout from your repo using stored credentials
        checkout([$class: 'GitSCM',
          branches: [[name: '*/main']],
          userRemoteConfigs: [[url: env.REPO, credentialsId: env.GIT_CRED]]
        ])
      }
    }

    stage('Install Dependencies') {
      steps {
        bat 'npm install > install.log 2>&1'
        archiveArtifacts artifacts: 'install.log', onlyIfSuccessful: false
      }
    }

    stage('Run Tests') {
      steps {
        // allow tests to fail but continue pipeline
        bat 'npm test > test.log 2>&1 || exit /b 0'
        archiveArtifacts artifacts: 'test.log', onlyIfSuccessful: false
      }
      post {
        always {
          // send email with test log attached
          emailext (
            to: env.NOTIFY_EMAIL,
            subject: "Test stage: ${currentBuild.currentResult} - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: "Test stage finished for ${env.JOB_NAME} #${env.BUILD_NUMBER}. See attached test.log",
            attachmentsPattern: 'test.log'
          )
        }
      }
    }

    stage('Generate Coverage Report') {
      steps {
        // if project has a coverage script
        bat 'npm run coverage > coverage.log 2>&1 || exit /b 0'
        archiveArtifacts artifacts: 'coverage.log', onlyIfSuccessful: false
      }
    }

    stage('NPM Audit (Security Scan)') {
      steps {
        // capture npm audit JSON (non-zero exit allowed)
        bat 'npm audit --json > audit.json || exit /b 0'
        archiveArtifacts artifacts: 'audit.json', onlyIfSuccessful: false
      }
      post {
        always {
          emailext (
            to: env.NOTIFY_EMAIL,
            subject: "Security scan: ${currentBuild.currentResult} - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: "Security scan finished for ${env.JOB_NAME} #${env.BUILD_NUMBER}. See attached audit.json",
            attachmentsPattern: 'audit.json'
          )
        }
      }
    }
  }

  post {
    always {
      // Final summary email with select attachments
      emailext (
        to: env.NOTIFY_EMAIL,
        subject: "Pipeline ${currentBuild.currentResult}: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        body: "Pipeline finished. Attached: test.log, audit.json (if present).",
        attachmentsPattern: 'test.log,audit.json'
      )
    }
  }
}
