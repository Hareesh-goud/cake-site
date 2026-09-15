pipeline {
    agent any
 
    options {
        skipDefaultCheckout()
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 30, unit: 'MINUTES')
        timestamps()
    }
 
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        DOCKER_REGISTRY = 'docker.io'
        DOCKER_CREDENTIALS_ID = 'docker-hub-credentials'
        IMAGE_NAME = 'govindhan1234/cake-site-2'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        AWS_REGION = 'us-east-1'
        EKS_CLUSTER_NAME = 'db4freash-cluster'
        AWS_CREDENTIALS_ID = 'aws-credentials'
        CI = 'true'
    }
 
    stages {
 
        stage('Git Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }
 
        stage('Install Dependencies') {
            steps {
                script {
                    def nodeHome = tool 'node'
                    env.PATH = "${nodeHome}/bin:${env.PATH}"
                }
                echo 'Installing dependencies...'
                sh 'npm install'
            }
        }
 
        stage('ESLint') {
            steps {
                script {
                    def nodeHome = tool 'node'
                    env.PATH = "${nodeHome}/bin:${env.PATH}"
                }
                echo 'Running ESLint code analysis...'
                sh 'npm run lint'
            }
        }
 
        stage('Tests') {
            steps {
                script {
                    def nodeHome = tool 'node'
                    env.PATH = "${nodeHome}/bin:${env.PATH}"
                }
                echo 'Running unit tests...'
                sh 'npm test -- --watchAll=false --coverage --passWithNoTests'
            }
        }
 
        stage('SonarQube Scan') {
            steps {
                echo 'Running SonarQube static code analysis...'
                withSonarQubeEnv('sonar-server') {
                    sh """
                        ${SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=cake-site \
                        -Dsonar.projectName=cake-site \
                        -Dsonar.sources=src \
                        -Dsonar.tests=src \
                        -Dsonar.test.inclusions=**/*.test.js,**/*.test.jsx \
                        -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info
                    """
                }
            }
        }
 
        stage('Quality Gate') {
            steps {
                echo 'Verifying SonarQube Quality Gate...'
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
 
        stage('Trivy FS Scan') {
            steps {
                echo 'Running Trivy File System vulnerability scan...'
                sh 'trivy fs . --severity HIGH,CRITICAL --exit-code 0'
            }
        }
 
        stage('Build') {
            steps {
                script {
                    def nodeHome = tool 'node'
                    env.PATH = "${nodeHome}/bin:${env.PATH}"
                }
                echo 'Building production application package...'
                sh 'npm run build'
            }
        }
    }
 
    post {
        always {
            echo 'CI Pipeline run finished.'
        }
        success {
            echo 'CI Pipeline completed successfully!'
        }
        failure {
            echo 'CI Pipeline failed! Please check logs.'
        }
    }
}
