pipeline {
  agent {
    kubernetes {
      yamlFile 'build-agent.yaml'
      defaultContainer 'maven'
      idleMinutes 1
    }
  }

  options {
    buildDiscarder(logRotator(numToKeepStr: '10'))
    timestamps()
    timeout(time: 30, unit: 'MINUTES')
  }

  environment {
    GITHUB_CREDENTIALS = credentials('Github-Cred')
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scmGit(
          branches: [[name: '*/main']],
          userRemoteConfigs: [[
            url: 'https://github.com/ujemeboi/dso-demo-new1.git',
            credentialsId: 'Github-Cred'
          ]]
        )
      }
    }

    stage('Build') {
      parallel {
        stage('Compile') {
          steps {
            container('maven') {
              sh 'mvn compile'
            }
          }
        }
      }
    }

    stage('Test') {
      parallel {
        stage('Unit Tests') {
          steps {
            container('maven') {
              sh 'mvn test'
            }
          }
          post {
            always {
              junit '**/target/surefire-reports/*.xml'
            }
          }
        }
      }
    }

    stage('Package') {
      parallel {
        stage('Create Jarfile') {
          steps {
            container('maven') {
              sh 'mvn package -DskipTests'
            }
          }
          post {
            success {
              archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
            }
          }
        }
      }
    }

    stage('Deploy to Dev') {
      steps {
        sh "echo done"
      }
    }
  }

  post {
    success {
      githubNotify(
        credentialsId: 'Github-Cred',
        status: 'SUCCESS',
        description: 'Build passed',
        context: 'ci/jenkins'
      )
    }
    failure {
      githubNotify(
        credentialsId: 'Githuib-Cred',
        status: 'FAILURE',
        description: 'Build failed',
        context: 'ci/jenkins'
      )
    }
    always {
      cleanWs()
    }
  }
}
