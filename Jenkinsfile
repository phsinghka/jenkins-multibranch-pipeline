pipeline {
    agent any 

    environment  {
        DOCKER_IMAGE = "phsinghka/baristaimage"
        CREDENTIALS_ID = 'docker-hub-credentials'
    }

    stages {
        stage ('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
    steps {
        script {
            if(env.BRANCH_NAME == 'dev'){
                echo 'Building for Development Environment.'
            }
            else if (env.BRANCH_NAME == 'staging') {
                echo 'Building for Staging Environment'
            }
            else if (env.BRANCH_NAME == 'prod') {
                echo 'Building for Production Environment'
            }

            // Construct the shell command properly
            def dockerBuildCommand = "docker build -t ${DOCKER_IMAGE}:${env.BRANCH_NAME} ."
            echo "Running: ${dockerBuildCommand}"
            sh dockerBuildCommand
        }
    }
}


        stage ('Test') {
            steps {
                echo "Running Test"
                sh './run-tests.sh'
            }
        }

        stage ('Push to Docker hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: CREDENTIALS_ID, usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh 'docker login -u $USERNAME -p $PASSWORD'
                    sh 'docker push ${DOCKER_IMAGE}:${env.BRANCH_NAME}'
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'dev') {
                        echo "Deploying to Development Environment"
                        sh './deploy-dev.sh' 
                    } else if (env.BRANCH_NAME == 'staging') {
                        echo "Deploying to Staging Environment"
                        sh './deploy-staging.sh' 
                    } else if (env.BRANCH_NAME == 'prod') {
                        echo "Deploying to Production Environment"
                        sh './deploy-prod.sh' 
                    }
                }
            }
        }


    }

    post {
        always {
            mail    to: 'phsinghka@gmail.com',
                    subject: "Jenkins Pipeline: ${currentBuild.fullDisplayName}",
                    body: "Build ${currentBuild.result}: ${env.BUILD_URL}"
        }
        success {
        slackSend channel: '#devops',
                  color: 'good',
                  message: "Build succeeded: ${env.JOB_NAME} - ${env.BUILD_NUMBER}\n${env.BUILD_URL}"
    }
    failure {
        slackSend channel: '#devops',
                  color: 'danger',
                  message: "Build failed: ${env.JOB_NAME} - ${env.BUILD_NUMBER}\n${env.BUILD_URL}"
    }


    }
}