pipeline {
    agent any

    environment {
        NEXUS_URL = 'nexus:8081/nexus'
        NEXUS_REPO = '11'
        DOCKER_IMAGE = 'docker_image/Dockerfile'
    }

    stages {
        stage('Checkout') {
            steps {
                git url: "${GIT_URL}", branch: 'master'
            }
        }


        stage('Compile') {
            steps {
                sh 'mvn clean package'
                echo "${GIT_COMMIT}"
            }
        }

        stage('Publish to Nexus') {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus2',
                    protocol: 'http',
                    nexusUrl: "${NEXUS_URL}",
                    groupId: 'com.hello_world',
                    version: "${BUILD_ID}-${GIT_COMMIT}",
                    repository: "${NEXUS_REPO}",
                    credentialsId: '2',
                    artifacts: [
                        [artifactId: 'hello_world_project', classifier: '', file: 'target/hello_world_project-1.3.jar', type: 'jar']
                    ]
                )
            }
        }


        stage('Build Docker Image') {
            steps {
                script {
                    def version = readFile("${VERSION_FILE}").trim()
                    docker.build("dockerfile:${version}", '-f ./docker_image/dockerfile .')
                }
            }
        }


        stage('Cleanup') {
            steps {
                cleanWs()
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}