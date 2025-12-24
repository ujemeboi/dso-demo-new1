pipeline {
  agent {
    kubernetes {
      yamlFile 'build-agent.yaml'
      defaultContainer 'maven'
      idleMinutes 1
    }
  }

  stages {
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
        }
      }
    }

    stage('Build Docker Image') {
      steps {
        container('kaniko') {
          sh '''
            /kaniko/executor \
              -f `pwd`/Dockerfile \
              -c `pwd` \
              --destination=docker.io/ernest633/dso-demo:v1 \
              --cache=true
          '''
        }
      }
    }

    stage('Deploy to Dev') {
      steps {
        sh "echo done"
      }
    }
  }
}

