pipeline {
    agent any

    environment {
        NEXUS_URL = 'nexus:8081/nexus'
        NEXUS_REPO = '11'
        DOCKER_IMAGE = 'docker_image/Dockerfile'
        VERSION_FILE = 'version.txt'
    }

    stages {
        stage('Checkout') {
            steps {
                git url: "${GIT_REPO}", branch: 'master'
            }
        }

        stage('Increment Version') {
            steps {
                script {
                    def version = readFile("${VERSION_FILE}").trim()
                    echo "Current Version: ${version}"
                    def (major, minor, patch) = version.tokenize('.')
                    patch = patch.toInteger() + 1
                    def newVersion = "${major}.${minor}.${patch}"
                    writeFile file: "${VERSION_FILE}", text: newVersion
                    echo "New Version: ${newVersion}"
                }
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Publish to Nexus') {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus2',
                    protocol: 'http',
                    nexusUrl: "${NEXUS_URL}",
                    groupId: 'com.hello_world',
                    version: readFile("${VERSION_FILE}").trim(),
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