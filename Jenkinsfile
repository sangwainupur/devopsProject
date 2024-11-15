pipeline {
    agent any

    tools {
        dockerTool 'docker' // Ensure 'docker' matches the name in Global Tool Configuration
    }

    environment {
        REGISTRY = 'deveshksh'                          // Docker Hub username
        REGISTRY_CREDENTIALS = 'docker-hub-credentials' // Docker Hub credentials ID in Jenkins
        PATH = "/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin"  // Ensure correct PATH
        DOCKER_CONFIG = "/root/.docker"                 // Docker config path, adjust if needed
    }

    stages {
        stage('Test Docker and Dotnet') {
            steps {
                sh 'docker --version'
                sh 'dotnet --version'
            }
        }
        stage('Checkout') {
            steps {
                git url: 'https://github.com/deveshksh/devopsProject.git', branch: 'master', credentialsId: 'github-credentials'
            }
        }

        stage('Build and Test') {
            steps {
                echo "Building and testing the code..."
                sh 'dotnet build store.sln'
                sh 'dotnet test tests/CartMicroservice.UnitTests/CartMicroservice.UnitTests.csproj'
                sh 'dotnet test tests/CatalogMicroservice.UnitTests/CatalogMicroservice.UnitTests.csproj'
                sh 'dotnet test tests/IdentityMicroservice.UnitTests/IdentityMicroservice.UnitTests.csproj'
            }
        }

        stage('Build and Push Docker Images') {
            parallel {
                stage('Catalog Microservice') {
                    steps {
                        script {
                            dockerBuildAndPush('catalog-microservice', 'src/microservices/CatalogMicroservice')
                        }
                    }
                }
                stage('Cart Microservice') {
                    steps {
                        script {
                            dockerBuildAndPush('cart-microservice', 'src/microservices/CartMicroservice')
                        }
                    }
                }
                stage('Identity Microservice') {
                    steps {
                        script {
                            dockerBuildAndPush('identity-microservice', 'src/microservices/IdentityMicroservice')
                        }
                    }
                }
                stage('Frontend Gateway') {
                    steps {
                        script {
                            dockerBuildAndPush('frontend-gateway', 'src/gateways/FrontendGateway')
                        }
                    }
                }
                stage('Backend Gateway') {
                    steps {
                        script {
                            dockerBuildAndPush('backend-gateway', 'src/gateways/BackendGateway')
                        }
                    }
                }
                stage('Frontend') {
                    steps {
                        script {
                            dockerBuildAndPush('frontend-microservice', 'src/uis/Frontend')
                        }
                    }
                }
                stage('Backend') {
                    steps {
                        script {
                            dockerBuildAndPush('backend-microservice', 'src/uis/Backend')
                        }
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    deployMicroservices()
                }
            }
        }
    }
}

def dockerBuildAndPush(serviceName, contextDir) {
    def imageName = "${env.REGISTRY}/${serviceName}:latest"
    withEnv(["PATH=${env.PATH}"]) {
        // Diagnostic commands
        sh 'echo "Current PATH: $PATH"'
        sh 'which docker'
        sh 'docker --version'
        
        docker.withRegistry('', env.REGISTRY_CREDENTIALS) {
            // Build the Docker image
            def app = docker.build(imageName, contextDir)
            // Push the Docker image to the registry
            app.push()
        }
    }
}

def deployMicroservices() {
    echo "Deploying microservices..."
    withEnv(["PATH=${env.PATH}"]) {
        sh '''
        docker-compose down
        docker-compose up -d
        '''
    }
}