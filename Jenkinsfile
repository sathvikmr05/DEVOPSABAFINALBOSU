pipeline {
    agent any

    tools {
        jdk 'jdk-17'               // Set this name in Jenkins > Global Tool Configuration
        maven 'maven-3.9.9'        // Set this name in Jenkins > Global Tool Configuration
    }

    environment {
        DOCKER_IMAGE = 'sathwikpadmanabha/devopsfinal'
        DOCKER_TAG = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                bat '''
                    echo "JAVA_HOME: %JAVA_HOME%"
                    echo "Maven version:"
                    mvn --version
                    echo "Building project..."
                    mvn clean package
                '''
            }
        }

        stage('Test') {
            steps {
                bat '''
                    echo "Running tests..."
                    mvn test
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                bat """
                    echo Building Docker image...
                    docker build -t ${env.DOCKER_IMAGE}:${env.DOCKER_TAG} .
                    docker tag ${env.DOCKER_IMAGE}:${env.DOCKER_TAG} ${env.DOCKER_IMAGE}:latest
                """
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    bat """
                        echo Logging into Docker Hub...
                        echo %DOCKER_PASSWORD% | docker login -u %DOCKER_USERNAME% --password-stdin
                        docker push ${env.DOCKER_IMAGE}:${env.DOCKER_TAG}
                        docker push ${env.DOCKER_IMAGE}:latest
                        docker logout
                    """
                }
            }
        }
    }

    post {
        always {
            emailext (
                subject: "Pipeline Status: ${currentBuild.fullDisplayName}",
                body: """<p>Pipeline Status: ${currentBuild.result}</p>
                         <p>Check console output at <a href='${BUILD_URL}'>${BUILD_URL}</a></p>""",
                mimeType: 'text/html',
                to: 'sathwikpadmanabha@gmail.com'
            )
        }
    }
}
