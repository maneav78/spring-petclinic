pipeline {
    agent any

    environment {
        REPO_MR = "mr"
        REPO_MAIN = "main"
        IMAGE_NAME = "spring-petclinic"
    }

    stages {
        stage ('Checkout') {
            when {
                expression { env.CHANGE_ID != null }
            }
            steps {
                checkout scm
            }
        }

        stage ('Checkstyle') {
            when {
                expression { env.CHANGE_ID != null }
            }
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
            when {
                expression { env.CHANGE_ID != null }
            }
            steps {
                script {
                    sh 'mvn test'
                }
            }
        }

        stage ('Build') {
            when {
                expression { env.CHANGE_ID != null }
            }
            steps {
                script {
                    sh 'mvn clean install -DskipTests'
                }
            }
        }

        stage ('Docker Image for MR') {
            when {
                expression { env.CHANGE_ID != null }
            }
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'docker-credentials',
                        usernameVariable: 'DOCKER_CREDS_USR',
                        passwordVariable: 'DOCKER_CREDS_PSW'
                    )]) {
                        sh """
                        echo "${DOCKER_CREDS_PSW}" | docker login localhost:8084 -u "${DOCKER_CREDS_USR}" --password-stdin
                        docker build -t ${IMAGE_NAME}:${GIT_COMMIT} .
                        docker tag ${IMAGE_NAME}:${GIT_COMMIT} localhost:8084/repository/${REPO_MR}/${IMAGE_NAME}:${GIT_COMMIT}
                        docker push localhost:8084/repository/${REPO_MR}/${IMAGE_NAME}:${GIT_COMMIT}
                        """
                    }
                }
            }
        }

        stage ('Docker Image for main branch') {
            when {
                expression {
                    env.BRANCH_NAME == 'origin/main' || env.GIT_BRANCH?.contains('main')
                }
            }
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'docker-credentials',
                        usernameVariable: 'DOCKER_CREDS_USR',
                        passwordVariable: 'DOCKER_CREDS_PSW'
                    )]) {
                        sh """
                        echo "${DOCKER_CREDS_PSW}" | docker login localhost:8085 -u "${DOCKER_CREDS_USR}" --password-stdin
                        docker build -t ${IMAGE_NAME}:${GIT_COMMIT} .
                        docker tag ${IMAGE_NAME}:${GIT_COMMIT} localhost:8085/repository/${REPO_MAIN}/${IMAGE_NAME}:${GIT_COMMIT}
                        docker push localhost:8085/repository/${REPO_MAIN}/${IMAGE_NAME}:${GIT_COMMIT}
                        """
                    }
                }
            }
        }
    }
}
