pipeline {
    agent any
    
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        DOCKER_IMAGE = 'sathwikpadmanabha/devopsfinal'
        DOCKER_TAG = "${BUILD_NUMBER}"
        MAVEN_HOME = 'C:\\Program Files\\Apache\\maven'
        PATH = "${MAVEN_HOME}\\bin;${env.PATH}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                script {
                    def mvnCmd = "C:\\Program Files\\Apache\\maven\\bin\\mvn.cmd"
                    bat """
                        echo Maven version:
                        "${mvnCmd}" --version
                        echo Building project...
                        "${mvnCmd}" clean package
                    """
                }
            }
        }
        
        stage('Test') {
            steps {
                script {
                    def mvnCmd = "C:\\Program Files\\Apache\\maven\\bin\\mvn.cmd"
                    bat """
                        echo Running tests...
                        "${mvnCmd}" test
                    """
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                bat """
                    echo Building Docker image...
                    docker build -t %DOCKER_IMAGE%:%DOCKER_TAG% .
                    docker tag %DOCKER_IMAGE%:%DOCKER_TAG% %DOCKER_IMAGE%:latest
                """
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    bat """
                        echo Logging into Docker Hub...
                        echo %DOCKER_PASSWORD% | docker login -u %DOCKER_USERNAME% --password-stdin
                        echo Pushing Docker images...
                        docker push %DOCKER_IMAGE%:%DOCKER_TAG%
                        docker push %DOCKER_IMAGE%:latest
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
                <p>Check console output at &QUOT;<a href='${BUILD_URL}'>${BUILD_URL}</a>&QUOT;</p>""",
                recipientProviders: [[$class: 'DevelopersRecipientProvider']],
                to: '${DEFAULT_RECIPIENTS}'
            )
        }
    }
} 
