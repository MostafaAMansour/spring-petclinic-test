pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = 'dockerhub-MostafaAMansour'
        DOCKER_HUB_REPO = 'mostafaamansour/spring_petclinic'
        GITHUB_REPO = 'https://github.com/MostafaAMansour/spring-petclinic.git'
        GITHUB_CREDENTIALS = 'Github-MostafaAMansour'
        BRANCH = 'main'
        RECIPIENT = 'toota353535@gmail.com'
        ARTIFACT_DIR = 'artifacts'
    }

    triggers {
        pollSCM '* * * * *'
    }

    stages {
        stage('Checkout') {
            steps {
                pollSCM '* * * * *'
        //         script {
        //             checkout([$class: 'GitSCM', branches: [[name: "${BRANCH}"]],
        //                 userRemoteConfigs: [[url: "${GITHUB_REPO}", credentialsId: "${GITHUB_CREDENTIALS}"]]
        //             ])
        //         }
            }
        }

        stage('Build') {
            steps {                
                script {
                    // Build the Docker images
                    sh 'docker compose -f spring-petclinic/docker-compose.yml -f spring-petclinic/docker-compose-dev.yml build'
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    // Start services
                    sh 'docker compose -f spring-petclinic/docker-compose.yml -f spring-petclinic/docker-compose-dev.yml up -d'
                    // Run the test command
                    sh 'sleep 30' // wait for the service to be ready
                    // Execute the test command inside the app service
                    sh 'docker compose -f spring-petclinic/docker-compose.yml -f spring-petclinic/docker-compose-dev.yml exec app curl localhost:8080/owners/11 | grep Mostafa'
                }
            }
        }
        stage('Deploy check installation') {
            agent {
                label 'ubuntu-docker-agent'
            }
            steps {
                script {
                    def dockerInstalled = sh(
                        script: 'command -v docker',
                        returnStatus: true
                    ) == 0
                    def gitInstalled = sh(
                        script: 'command -v git',
                        returnStatus: true
                    ) == 0
                    if (!dockerInstalled) {
                        sh 'apt update'
                        sh 'sudo apt install apt-transport-https ca-certificates curl software-properties-common'
                        sh 'curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -'
                        sh 'sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"'
                        sh 'apt update'
                        sh 'apt-cache policy docker-ce'
                        sh 'sudo apt install docker-ce'
                    }
                    if(!gitInstalled){
                        sh 'sudo apt update'
                        sh 'sudo apt install git-all'
                    }
                }
            }
        }
        stage('deploy run') {
            agent {
                label 'ubuntu-docker-agent'
            }
            when {
                expression {
                    sh(script: 'command -v docker', returnStatus: true) == 0
                }
            }
            steps {
                script {
                    // Cloning the reposatory
                    sh 'git clone https://github.com/MostafaAMansour/spring-petclinic-aws'
                    // Start services
                    sh 'docker compose -f spring-petclinic/docker-compose.yml -f spring-petclinic/docker-compose-dev.yml up -d'
                }
            }
        }
        stage('Archive Artifacts') {
            steps {
                script {
                    // Create artifact directory if it doesn't exist
                    sh "mkdir -p ${ARTIFACT_DIR}"
                    // Copy artifacts from the container to the host
                    sh 'docker cp $(docker compose -f spring-petclinic/docker-compose.yml -f spring-petclinic/docker-compose-dev.yml ps -q app):/app/ ${ARTIFACT_DIR}'
                }
                archiveArtifacts artifacts: "${ARTIFACT_DIR}/app/*.jar", allowEmptyArchive: true
            }
        }
        stage('Push Docker Images') {
            steps {
                script {
                    // Push the built images to Docker Hub
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_HUB_CREDENTIALS}", usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        sh """
                        echo "${PASSWORD}" | docker login -u "${USERNAME}" --password-stdin
                        docker compose -f spring-petclinic/docker-compose.yml -f spring-petclinic/docker-compose-dev.yml push
                        docker logout
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            mail subject: "SUCCESS: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                     body: "Good news! Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' succeeded.",
                     to: "${RECIPIENT}"
        }
        failure {
            mail subject: "FAILURE: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                     body: "Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' failed. Please check the logs for details.",
                     to: "${RECIPIENT}"
        }
        cleanup {
            script {
                // List and remove containers, networks, images, and volumes specific to the project
                sh 'docker compose -f spring-petclinic/docker-compose.yml -f spring-petclinic/docker-compose-dev.yml down'
            }
            // Clean up workspace and project-specific Docker containers, networks, images, and volumes
            cleanWs()
        }
    }
}
