pipeline{
  agent any
  stages{
    stage('Checkout Code'){
      steps{
        git 'https://github.com/jzeleke/MSIS4363_Assignment6'
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
