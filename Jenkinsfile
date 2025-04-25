pipeline {
    agent any
    
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
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
                powershell '''
                    Write-Host "Maven version:"
                    mvn --version
                    Write-Host "Building project..."
                    mvn clean package
                '''
            }
        }
        
        stage('Test') {
            steps {
                powershell '''
                    Write-Host "Running tests..."
                    mvn test
                '''
            }
        }
        
        stage('Build Docker Image') {
            steps {
                powershell '''
                    Write-Host "Building Docker image..."
                    docker build -t $env:DOCKER_IMAGE:$env:DOCKER_TAG .
                    docker tag $env:DOCKER_IMAGE:$env:DOCKER_TAG $env:DOCKER_IMAGE:latest
                '''
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    powershell '''
                        Write-Host "Logging into Docker Hub..."
                        $env:DOCKER_PASSWORD | docker login -u $env:DOCKER_USERNAME --password-stdin
                        Write-Host "Pushing Docker images..."
                        docker push $env:DOCKER_IMAGE:$env:DOCKER_TAG
                        docker push $env:DOCKER_IMAGE:latest
                    '''
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
