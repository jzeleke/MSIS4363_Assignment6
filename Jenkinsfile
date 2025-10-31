pipeline {
  agent any
  options { timestamps() }

  // Choose ONE trigger below in your job config or here:
  // triggers { githubPush() }       // if Jenkins is reachable by GitHub webhook
  // triggers { pollSCM('H/2 * * * *') } // if Jenkins is local; polls every ~2 minutes

  environment {
    ARTIFACT   = 'app.zip'
    STAGING_URL = 'https://StagingAssignment6.azurewebsites.net'
    PROD_URL    = 'https://ProductionAssignment6.azurewebsites.net'
    HEALTH_PATH = '/'   // change to '/health' if you have a health endpoint
  }

  stages {
    stage('Checkout Code') {
      steps {
        // If this pipeline is defined from the repo (Jenkinsfile in SCM), use:
        checkout scm
        // If you are pasting pipeline in Jenkins UI instead, use the next line instead of checkout scm:
        // git 'https://github.com/bauerb2525/MSIS4363Assignment6_Integration'
      }
    }

    stage('Build (.NET)') {
      steps {
        bat 'dotnet restore'
        bat 'dotnet build --configuration Release'
        bat 'dotnet publish --configuration Release -o publish'
      }
    }

    stage('Package') {
      steps {
        powershell 'Compress-Archive -Path publish\\* -DestinationPath app.zip -Force'
        archiveArtifacts artifacts: 'app.zip', fingerprint: true
      }
    }

    // --- Deploy to STAGING whenever we build the staging branch ---
    stage('Deploy to STAGING') {
      when {
        anyOf {
          branch 'staging'
          branch 'Staging'
        }
      }
      steps {
        copyArtifacts(projectName: env.JOB_NAME, selector: lastSuccessful())
        withCredentials([string(credentialsId: 'AZURE_PUBPROFILE_STAGING_ASSIGN6', variable: 'PUBXML')]) {
          powershell '''
$xml  = [xml]$env:PUBXML
$user = $xml.publishData.publishProfile.userName
$pass = $xml.publishData.publishProfile.userPWD
$kudu = ($xml.publishData.publishProfile.publishUrl -replace ':443','') + '/api/zipdeploy'
Invoke-RestMethod -Uri $kudu -Method POST -InFile "app.zip" -ContentType "application/zip" `
  -Authentication Basic -UserName $user -Password $pass
'''
        }
      }
    }

    stage('Smoke Test (staging)') {
      when {
        anyOf { branch 'staging'; branch 'Staging' }
      }
      steps {
        powershell '''
$u = "${env:STAGING_URL}${env:HEALTH_PATH}"
$r = Invoke-WebRequest $u -UseBasicParsing -TimeoutSec 20
if ($r.StatusCode -ne 200) { throw "Staging health check failed: $($r.StatusCode)" }
'''
      }
    }

    // --- Promote and deploy to PRODUCTION when building main/production ---
    stage('Approve production deploy') {
      when {
        anyOf { branch 'main'; branch 'production'; branch 'Production' }
      }
      steps {
        input message: 'Deploy to PRODUCTION (ProductionAssignment6)?', ok: 'Deploy'
      }
    }

    stage('Deploy to PRODUCTION') {
      when {
        anyOf { branch 'main'; branch 'production'; branch 'Production' }
      }
      steps {
        copyArtifacts(projectName: env.JOB_NAME, selector: lastSuccessful())
        withCredentials([string(credentialsId: 'AZURE_PUBPROFILE_PROD_ASSIGN6', variable: 'PUBXML')]) {
          powershell '''
$xml  = [xml]$env:PUBXML
$user = $xml.publishData.publishProfile.userName
$pass = $xml.publishData.publishProfile.userPWD
$kudu = ($xml.publishData.publishProfile.publishUrl -replace ':443','') + '/api/zipdeploy'
Invoke-RestMethod -Uri $kudu -Method POST -InFile "app.zip" -ContentType "application/zip" `
  -Authentication Basic -UserName $user -Password $pass
'''
        }
      }
    }
  }

  post {
    success { bat 'echo "build successful"' }
    failure { bat 'echo "build failed"' }
  }
}
