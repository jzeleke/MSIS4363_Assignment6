pipeline{
  agent any
  stages{
    stage('Checkout Code'){
      steps{
        git 'https://github.com/bauerb2525/MSIS4363Assignment6_Integration'
      }
    }
    stage('Build'){
      steps{
        bat 'echo "building the app"'
      }
    }
    stage('Test'){
      steps{
        bat 'echo "Running Tests"'
      }
    }
    stage('Deploy'){
      steps{
        bat 'echo "deploying"'
      }
    }
  }
  post{
    success{
      bat 'echo "build successful"'
    }
    failure{
      bat 'echo "build failed"'
    }
  }
}
