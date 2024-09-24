pipeline {
    agent {
      label 'Docker-java-agent'
    }

    environment {
        DOCKER_HUB_CREDENTIALS = 'dockerhub-MostafaAMansour'
        DOCKER_HUB_REPO = 'mostafaamansour/spring_petclinic'
        GITHUB_REPO = 'https://github.com/MostafaAMansour/spring-petclinic-test.git'
        GITHUB_CREDENTIALS = 'Github-MostafaAMansour'
        BRANCH = 'main'
        RECIPIENT = 'toota353535@gmail.com'
        ARTIFACT_DIR = 'artifacts'
    }

    //triggers {
    //    pollSCM '* * * * *'
    //}

    stages {
        stage('Checkout') {
            steps {
                script {
                    checkout([$class: 'GitSCM', branches: [[name: "${BRANCH}"]],
                        userRemoteConfigs: [[url: "${GITHUB_REPO}", credentialsId: "${GITHUB_CREDENTIALS}"]]
                    ])
                }
            }
        }

        stage('Build') {
            steps {                
                script {
                    // change permission
                    sh 'chmod +x ./mvnw'
                    // Build the maven
                    sh './mvnw package'
                }
            }
        }

        // stage('Test') {
        //     steps {
        //         script {
        //             // Start services
        //             sh 'java -jar target/*.jar'
        //             // Run the test command
        //             sh 'sleep 30' // wait for the service to be ready
        //             // Execute the test command inside the app service
        //             sh 'curl localhost:8080/owners/10'
        //         }
        //     }
        // }

        // stage('Archive Artifacts') {
        //     steps {
        //         script {
        //             // Create artifact directory if it doesn't exist
        //             sh "mkdir -p ${ARTIFACT_DIR}"
        //         }
        //         archiveArtifacts artifacts: "target/*.jar", allowEmptyArchive: true
        //     }
        // }
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
        // cleanup {
        //     // Clean up workspace and project-specific Docker containers, networks, images, and volumes
        //     cleanWs()
        // }
    }
}