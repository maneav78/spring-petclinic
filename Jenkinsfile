pipeline {
    agent any

    environment {
        REPO_MR = "mr"
        REPO_MAIN = "main"
        IMAGE_NAME = "spring-petclinic"
    }

    stages {
        stage ('Checkout') {
            steps {
                checkout scm 
            }
        }

        stage ('Checkstyle') {
            steps {
                script {
                    sh 'mvn checkstyle:checkstyle'
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: '**/checkstyle-result.xml'
                }
            }
        }

        stage ('Test') {
            steps {
                script {
                    sh 'mvn test'
                }
            }
        }

        stage ('Build') {
            steps {
                script {
                    sh 'mvn clean install -DskipTests'
                }
            }
        }

        stage ('Docker Image for MR') {
            when {
                branch 'PR-*'
            }
            steps {
                script {
                    sh """
                    docker build -t ${IMAGE_NAME}:${GIT_COMMIT} .
                    docker tag ${IMAGE_NAME}:${GIT_COMMIT} localhost:8084/repository/${REPO_MR}/${IMAGE_NAME}:${GIT_COMMIT}
                    docker push localhost:8084/repository/${REPO_MR}/${IMAGE_NAME}:${GIT_COMMIT}
                    """
                }
            }
        }

        stage ('Docker Image for main branch') {
            when {
                branch 'main'
            }
            steps {
                script {
                    sh """
                    docker build -t ${IMAGE_NAME}:${GIT_COMMIT} .
                    docker tag ${IMAGE_NAME}:${GIT_COMMIT} localhost:8084/repository/${REPO_MAIN}/${IMAGE_NAME}:${GIT_COMMIT}
                    docker push localhost:8084/repository/${REPO_MR}/${IMAGE_NAME}:${GIT_COMMIT}
                    """
                }
            }
        }
    }
}