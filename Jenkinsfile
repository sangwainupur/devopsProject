pipeline {
    agent any

    environment {
        REGISTRY = 'deveshksh'                          // Docker Hub username
        REGISTRY_CREDENTIALS = 'docker-hub-credentials' // Docker Hub credentials ID in Jenkins
        PATH = "/usr/local/bin:/opt/homebrew/bin:/usr/bin:/bin:/usr/sbin:/sbin" // Add docker and dotnet paths here
        DOCKER_CONFIG = "/root/.docker"                 // Docker config path, adjust if needed
    }

    stages {
        stage('Test Docker and Dotnet') {
            steps {
                sh '${DOCKER_PATH} --version'
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
                        withEnv(["DOCKER_PATH=${DOCKER_PATH}"]) {
                            dockerBuildAndPush('catalog-microservice', 'src/microservices/CatalogMicroservice/Dockerfile')
                        }
                    }
                }
                stage('Cart Microservice') {
                    steps {
                        withEnv(["DOCKER_PATH=${DOCKER_PATH}"]) {
                            dockerBuildAndPush('cart-microservice', 'src/microservices/CartMicroservice/Dockerfile')
                        }
                    }
                }
                stage('Identity Microservice') {
                    steps {
                        withEnv(["DOCKER_PATH=${DOCKER_PATH}"]) {
                            dockerBuildAndPush('identity-microservice', 'src/microservices/IdentityMicroservice/Dockerfile')
                        }
                    }
                }
                stage('Frontend Gateway') {
                    steps {
                        withEnv(["DOCKER_PATH=${DOCKER_PATH}"]) {
                            dockerBuildAndPush('frontend-gateway', 'src/gateways/FrontendGateway/Dockerfile')
                        }
                    }
                }
                stage('Backend Gateway') {
                    steps {
                        withEnv(["DOCKER_PATH=${DOCKER_PATH}"]) {
                            dockerBuildAndPush('backend-gateway', 'src/gateways/BackendGateway/Dockerfile')
                        }
                    }
                }
                stage('Frontend') {
                    steps {
                        withEnv(["DOCKER_PATH=${DOCKER_PATH}"]) {
                            dockerBuildAndPush('frontend-microservice', 'src/uis/Frontend/Dockerfile')
                        }
                    }
                }
                stage('Backend') {
                    steps {
                        withEnv(["DOCKER_PATH=${DOCKER_PATH}"]) {
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
    sh '''
    ${DOCKER_PATH}-compose down
    ${DOCKER_PATH}-compose up -d
    '''
}