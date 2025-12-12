pipeline {
    agent any

    environment {
        SONAR_TOKEN = credentials('SONAR_TOKEN')
        SONAR_HOST_URL = 'http://192.168.0.164:9000'
    }

    stages {

        stage('Clone Repository') {
            steps {
                sh 'rm -rf sonarqube-jenkins-demo || true'
                sh 'git clone https://github.com/iam-vinod/sonarqube-jenkins-demo.git'
                sh 'ls -l'
            }
        }

        stage('Build & Test') {
            steps {
                dir('sonarqube-jenkins-demo') {
                    sh 'mvn clean test'
                }
            }
            post {
                always {
                    dir('sonarqube-jenkins-demo') {
                        junit 'target/surefire-reports/*.xml'
                    }
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                dir('sonarqube-jenkins-demo') {
                    withSonarQubeEnv('MySonar') {
                        sh """
                            mvn sonar:sonar \
                                -Dsonar.projectKey=sonarqube-jenkins-demo \
                                -Dsonar.projectName=SonarJenkinsDemo \
                                -Dsonar.host.url=${SONAR_HOST_URL} \
                                -Dsonar.login=${SONAR_TOKEN}
                        """
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    def qg = waitForQualityGate()
                    if (qg.status != 'OK') {
                        error "❌ Quality Gate Failed: ${qg.status}"
                    }
                }
            }
        }
    }
}

