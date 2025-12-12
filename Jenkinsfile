pipeline {
    agent any

    environment {
        SONAR_HOST = "http://192.168.0.164:9000"
        SONAR_TOKEN = "squ_7ea8dacdd33aa8861b509fe582435a06cd94955e"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/iam-vinod/sonarqube-jenkins-demo.git'
            }
        }

        stage('Build & Test') {
            steps {
                sh '''
                    docker run --rm \
                      -v "$PWD":/app \
                      -v "$HOME/.m2":/root/.m2 \
                      -w /app \
                      maven:3.9.0-openjdk-17 \
                      mvn -B -DskipTests=false clean test
                '''
                junit '**/target/surefire-reports/*.xml'
            }
        }

        stage('Sonar Analysis') {
            steps {
                // Must match the name configured in Jenkins → SonarQube servers
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        docker run --rm \
                          -v "$PWD":/app \
                          -v "$HOME/.m2":/root/.m2 \
                          -w /app \
                          maven:3.9.0-openjdk-17 \
                          mvn -B sonar:sonar \
                          -Dsonar.projectKey=sonarqube-jenkins-demo \
                          -Dsonar.host.url=${SONAR_HOST} \
                          -Dsonar.login=${SONAR_TOKEN}
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
        always {
            archiveArtifacts artifacts: '**/target/*.jar', allowEmptyArchive: true
        }
    }
}

