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
        DOCKER_IMAGE = 'muayadoh/webapp:test'
        VM_IP = '192.168.100.124'
        VM_USER = 'user'
    }
    stages {
        stage('Build & Push') {
            steps {
                script {
                    // Injecting DOCKER_CONTEXT=default forces Docker to bypass the missing temporary context folder
                    withEnv(['DOCKER_CONTEXT=default']) {
                        docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-creds') {
                            def app = docker.build("${DOCKER_IMAGE}")
                            app.push()
                        }
                    }
                }
            }
        }
        stage('Deploy to VM') {
            steps {
                // Changed from 'sh' to 'bat' because your Jenkins agent runs on Windows
                sshagent(['vm-ssh-creds']) {
                    bat """
                    ssh -o StrictHostKeyChecking=no ${VM_USER}@${VM_IP} "docker pull ${DOCKER_IMAGE} && docker stop webapp || true && docker rm webapp || true && docker run -d -p 8080:8080 --name webapp ${DOCKER_IMAGE}"
                    """
                }
            }
        }
    }
}
