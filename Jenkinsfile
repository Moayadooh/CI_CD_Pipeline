// IIS
// pipeline {
//     agent any

//     environment {
//         // Adjust these to match your project's structure
//         PROJECT_PATH = 'WebApp/WebApp.csproj'
//         PUBLISH_DIR = 'publish_output'
//     }

//     stages {
//         stage('Restore') {
//             steps {
//                 echo 'Restoring NuGet packages...'
//                 bat "dotnet restore ${PROJECT_PATH}"
//             }
//         }

//         stage('Build') {
//             steps {
//                 echo 'Building the .NET 10 app...'
//                 bat "dotnet build ${PROJECT_PATH} --configuration Release --no-restore"
//             }
//         }

//         stage('Publish') {
//             steps {
//                 echo 'Publishing the application...'
//                 bat "dotnet publish ${PROJECT_PATH} --configuration Release --no-build --output ${PUBLISH_DIR}"
//             }
//         }

//         stage('Archive Artifacts') {
//             steps {
//                 // Saves the compiled output in Jenkins for download or later deployment
//                 archiveArtifacts artifacts: "${PUBLISH_DIR}/**/*", fingerprint: true
//             }
//         }

//         stage('Deploy') {
//             steps {
//                 echo 'Artifacts are ready. Deployment logic goes here.'
//                 // Example: Copy files to a server, build a Docker image, etc.
//             }
//         }
//     }

//     post {
//         always {
//             cleanWs() // Clean up the workspace after the build to save disk space
//         }
//         success {
//             echo 'Pipeline completed successfully!'
//         }
//         failure {
//             echo 'Pipeline failed. Check the Jenkins console output.'
//         }
//     }
// }

// Docker
pipeline {
    agent any
    environment {
        // Define your local registry address (Change 5000 if your registry uses a different port)
        REGISTRY_URL = 'localhost:5000'
        IMAGE_NAME   = 'webapp'
        DOCKER_IMAGE = "${REGISTRY_URL}/${IMAGE_NAME}"
        
        VM_IP        = '192.168.100.124'
        VM_USER      = 'user'
    }
    stages {
        stage('Build & Push to Local Registry') {
            steps {
                script {
                    withEnv([
                        'DOCKER_CONTEXT=default',
                        'DOCKER_HOST=npipe:////./pipe/docker_engine',
                        'DOCKER_BUILDKIT=1',
                        'BUILDKIT_PROGRESS=plain'
                    ]) {
                        bat """
                        docker build -t %DOCKER_IMAGE%:latest .
                        docker push %DOCKER_IMAGE%:latest
                        """
                    }
                }
            }
        }
        stage('Deploy to VM') {
            steps {
                sshagent(['vm-ssh-creds']) {
                    bat """
                    ssh -o StrictHostKeyChecking=no ${VM_USER}@${VM_IP} "docker pull ${DOCKER_IMAGE}:latest && docker stop webapp || true && docker rm webapp || true && docker run -d -p 8080:8080 --name webapp ${DOCKER_IMAGE}:latest"
                    """
                }
            }
        }
    }
}

