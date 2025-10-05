pipeline {
  agent any

  tools {
    // name must match the NodeJS installation name you created in Jenkins global tools
    nodejs "node-lts"
  }

  environment {
    REPO = "https://github.com/your_github_username/8.2CDevSecOps.git"
    JOB_EMAIL = "dev@example.com"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout([$class: 'GitSCM',
          branches: [[name: '*/main']],
          userRemoteConfigs: [[url: env.REPO, credentialsId: 'github-pat-cred']]])
      }
    }

    stage('Install Dependencies') {
      steps {
        // run npm install and save output to a log
        bat label: 'Install deps', script: 'npm install > install.log 2>&1'
        archiveArtifacts artifacts: 'install.log', onlyIfSuccessful: false
      }
    }

    stage('Run Tests') {
      steps {
        // run tests but do not fail the pipeline if tests fail (allow pipeline to continue)
        bat label: 'Run tests', script: 'npm test > test.log 2>&1 || exit /b 0'
        archiveArtifacts artifacts: 'test.log', onlyIfSuccessful: false
      }
      post {
        always {
          // send email for test stage with test.log attached
          emailext(
            to: env.JOB_EMAIL,
            subject: "Jenkins: Test stage - ${env.JOB_NAME} #${env.BUILD_NUMBER} - ${currentBuild.currentResult}",
            body: "Test stage finished for ${env.JOB_NAME} build #${env.BUILD_NUMBER}. See attached test log.",
            attachmentsPattern: 'test.log'
          )
        }
      }
    }

    stage('Generate Coverage Report') {
      steps {
        // if project has coverage script
        bat label: 'Coverage', script: 'npm run coverage > coverage.log 2>&1 || exit /b 0'
        archiveArtifacts artifacts: 'coverage.log', onlyIfSuccessful: false
      }
    }

    stage('NPM Audit (Security Scan)') {
      steps {
        // npm audit produces non-zero exit when vulnerabilities found — capture JSON output
        bat label: 'npm audit', script: 'npm audit --json > audit.json || exit /b 0'
        archiveArtifacts artifacts: 'audit.json', onlyIfSuccessful: false
      }
      post {
        always {
          // send email for security stage with audit.json attached
          emailext(
            to: env.JOB_EMAIL,
            subject: "Jenkins: Security scan - ${env.JOB_NAME} #${env.BUILD_NUMBER} - ${currentBuild.currentResult}",
            body: "Security scan (npm audit) finished for ${env.JOB_NAME} build #${env.BUILD_NUMBER}. See attached audit.json.",
            attachmentsPattern: 'audit.json'
          )
        }
      }
    }

    stage('Deploy to Staging') {
      steps {
        // example: use a script or Ansible playbook for deployments (must be available on Jenkins machine)
        // For Windows agent you might invoke a PowerShell deployment script
        bat label: 'Deploy to staging', script: 'powershell -File .\\scripts\\deploy-to-staging.ps1 > deploy-staging.log 2>&1'
        archiveArtifacts artifacts: 'deploy-staging.log', onlyIfSuccessful: false
      }
    }

    stage('Integration Tests on Staging') {
      steps {
        // run Newman (Postman) or other tests; save results to file
        bat label: 'Integration tests', script: 'newman run ./postman_collection.json --reporters cli,junit --reporter-junit-export integration-results.xml || exit /b 0'
        archiveArtifacts artifacts: 'integration-results.xml', onlyIfSuccessful: false
      }
    }

    stage('Deploy to Production') {
      steps {
        // example: call a script that triggers production deploy (CodeDeploy / Terraform apply etc.)
        bat label: 'Deploy to production', script: 'powershell -File .\\scripts\\deploy-to-production.ps1 > deploy-prod.log 2>&1'
        archiveArtifacts artifacts: 'deploy-prod.log', onlyIfSuccessful: false
      }
    }
  }

  post {
    always {
      // final email summary (optional)
      emailext(
        to: env.JOB_EMAIL,
        subject: "Jenkins: Build ${currentBuild.currentResult} - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        body: """Build ${currentBuild.currentResult} for ${env.JOB_NAME} #${env.BUILD_NUMBER}.
See archived artifacts and logs in the Jenkins build page.""",
        attachmentsPattern: 'test.log,audit.json,coverage.log'
      )
    }
  }
}
