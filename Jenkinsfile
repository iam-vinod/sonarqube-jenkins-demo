pipeline {
  agent any

  environment {
    SONAR_SERVER = 'LocalSonar'
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Build & Test') {
      steps {
        sh '''
          docker run --rm -v "$PWD":/app -v "$HOME/.m2":/root/.m2 -w /app \
            maven:3.9.0-openjdk-17 mvn -B -DskipTests=false clean test
        '''
        junit '**/target/surefire-reports/*.xml'
      }
    }

    stage('Sonar Analysis') {
      environment {
        SONAR_TOKEN = credentials('sonar-token')
      }
      steps {
        withSonarQubeEnv("${SONAR_SERVER}") {
          sh '''
            docker run --rm -v "$PWD":/app -v "$HOME/.m2":/root/.m2 -w /app \
              maven:3.9.0-openjdk-17 mvn -B sonar:sonar -Dsonar.login=${SONAR_TOKEN}
          '''
        }
      }
    }

    stage('Quality Gate') {
      steps {
        timeout(time: 5, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
        }
      }
    }
  }

  post {
    always { archiveArtifacts artifacts: '**/target/*.jar', allowEmptyArchive: true }
  }
}

