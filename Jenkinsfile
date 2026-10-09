pipeline {
    agent any

    options {
        skipDefaultCheckout()
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        DOCKER_REGISTRY = 'docker.io'
        DOCKER_CREDENTIALS_ID = 'docker-hub'
        IMAGE_NAME = 'harishgoud136/cake-site'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
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

                echo 'Running ESLint...'
                sh 'npm run lint'
            }
        }

        stage('Tests') {
            steps {
                script {
                    def nodeHome = tool 'node'
                    env.PATH = "${nodeHome}/bin:${env.PATH}"
                }

                echo 'Running tests...'
                sh 'npm test -- --watchAll=false'
            }
        }

        stage('SonarQube Scan') {
            steps {
                echo 'Skipping SonarQube Scan because server is down...'

                // withSonarQubeEnv('sonar-server') {
                //     withCredentials([
                //         string(
                //             credentialsId: 'sonar-token',
                //             variable: 'SONAR_TOKEN'
                //         )
                //     ]) {
                //         sh "${SCANNER_HOME}/bin/sonar-scanner -Dsonar.login=\$SONAR_TOKEN"
                //     }
                // }
            }
        }

        stage('Quality Gate') {
            steps {
                echo 'Skipping Quality Gate because SonarQube is down...'

                // waitForQualityGate abortPipeline: true
            }
        }

        stage('Trivy FS Scan') {
            steps {
                echo 'Skipping Trivy File System Scan per user request...'

                // sh 'trivy fs . --severity HIGH,CRITICAL'
            }
        }

        stage('Docker Build') {
            steps {
                echo 'Building Docker Image...'

                sh """
                    docker build \
                    -t ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} .
                """
            }
        }

        stage('Trivy Image Scan') {
            steps {
                echo 'Skipping Docker Image Scan per user request...'

                // sh "trivy image ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} --severity HIGH,CRITICAL"
            }
        }

        stage('Docker Push') {
            steps {
                echo 'Pushing Docker Image to Registry...'

                withCredentials([
                    usernamePassword(
                        credentialsId: env.DOCKER_CREDENTIALS_ID,
                        passwordVariable: 'DOCKER_PASSWORD',
                        usernameVariable: 'DOCKER_USERNAME'
                    )
                ]) {

                    sh """
                        echo \$DOCKER_PASSWORD | docker login \
                        ${DOCKER_REGISTRY} \
                        -u \$DOCKER_USERNAME \
                        --password-stdin
                    """

                    sh """
                        docker push \
                        ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Docker RUN') {
            steps {
                echo 'Building Docker Container...'

                sh """
                    docker run \
                    -d -p 80:80 --name cake  ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} 
                """
            }
        }

    }

    post {

        always {
            echo 'Pipeline execution completed.'
        }

        success {
            echo 'Success: The pipeline finished successfully!'
        }

        failure {
            echo 'Failure: The pipeline failed.'
        }
    }
}
