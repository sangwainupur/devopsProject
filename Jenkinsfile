pipeline {
    agent any

    environment {
        REGISTRY = 'deveshksh'                          // Docker Hub username
        REGISTRY_CREDENTIALS = 'docker-hub-credentials' // Docker Hub credentials ID in Jenkins
        PATH = "/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin"  // Ensure correct path for dotnet and docker
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
                // Pull code from the GitHub repository
                git url: 'https://github.com/deveshksh/devopsProject.git', branch: 'master'
            }
        }

        stage('Build and Test') {
            steps {
                script {
                    // Building the solution
                    echo "Building and testing the code..."
                    sh '/opt/homebrew/bin/dotnet build store.sln'
                    
                    // Run each test project individually
                    sh '/opt/homebrew/bin/dotnet test tests/CartMicroservice.UnitTests/CartMicroservice.UnitTests.csproj'
                    sh '/opt/homebrew/bin/dotnet test tests/CatalogMicroservice.UnitTests/CatalogMicroservice.UnitTests.csproj'
                    sh '/opt/homebrew/bin/dotnet test tests/IdentityMicroservice.UnitTests/IdentityMicroservice.UnitTests.csproj'
                }
            }
        }

        stage('Build and Push Docker Images') {
            parallel {
                stage('Catalog Microservice') {
                    steps {
                        script {
                            dockerBuildAndPush('catalog-microservice', 'src/microservices/CatalogMicroservice/Dockerfile')
                        }
                    }
                }
                stage('Cart Microservice') {
                    steps {
                        script {
                            dockerBuildAndPush('cart-microservice', 'src/microservices/CartMicroservice/Dockerfile')
                        }
                    }
                }
                stage('Identity Microservice') {
                    steps {
                        script {
                            dockerBuildAndPush('identity-microservice', 'src/microservices/IdentityMicroservice/Dockerfile')
                        }
                    }
                }
                stage('Frontend Gateway') {
                    steps {
                        script {
                            dockerBuildAndPush('frontend-gateway', 'src/gateways/FrontendGateway/Dockerfile')
                        }
                    }
                }
                stage('Backend Gateway') {
                    steps {
                        script {
                            dockerBuildAndPush('backend-gateway', 'src/gateways/BackendGateway/Dockerfile')
                        }
                    }
                }
                stage('Frontend') {
                    steps {
                        script {
                            dockerBuildAndPush('frontend-microservice', 'src/uis/Frontend/Dockerfile')
                        }
                    }
                }
                stage('Backend') {
                    steps {
                        script {
                            dockerBuildAndPush('backend-microservice', 'src/uis/Backend/Dockerfile')
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

def dockerBuildAndPush(serviceName, dockerfilePath) {
    def imageName = "${env.REGISTRY}/${serviceName}:latest"
    docker.withRegistry('', env.REGISTRY_CREDENTIALS) {
        def app = docker.build(imageName, "-f ${dockerfilePath} .")
        app.push()
    }
}

def deployMicroservices() {
    echo "Deploying microservices..."
    // Example: Deploy using Docker Compose or custom deploy script
    sh '''
    docker-compose down
    docker-compose up -d
    '''
}