pipeline{
    agent{
        label "jenkins-agent"
    }
    tools{
        jdk 'Java17'
        maven 'Maven3'
    }
    environment{
        APP_NAME = "complete-prodcution-e2e-pipeline"
        RELEASE = "2.0.0"
        DOCKER_USER = "asirabdelhady"
        DOCKER_PASS = 'dockerhub'
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        JENKINS_API_TOKEN = credentials('JENKINS_API_TOKEN')
    }
    stages{
        stage("Cleanup Workspace"){
            steps{
                cleanWs()
            }
        }

        stage("Checkout from SCM"){
            steps{
                git branch: 'main', credentialsId: 'github', url: "https://github.com/asirabdelhady/complete-prodcution-e2e-pipeline.git"
            }
        }

        stage("Build App"){
            steps{
                sh "mvn clean package"
            }
        }

        stage("Test App"){
            steps{
                sh "mvn test"
            }
        }
        
        stage("Sonarqube Analysis"){
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token'){
                    sh "mvn sonar:sonar"
                    }
                }
            }
        }

        stage("Quality Gate"){
            steps{
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
                }
            }
        }
        stage("Build and Push Docker Image"){
            steps{
                script{
                    
                    docker.withRegistry('',DOCKER_PASS){
                        docker_image = docker.build "${IMAGE_NAME}"
                    }

                    docker.withRegistry('',DOCKER_PASS){
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }
        }
        stage("Trigger CD Pipeline"){
            steps{
                script{
                    sh "curl -v -k --user asir:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'https://jenkins-server.com/job/gitops-complete-pipeline/buildWithParameters?token=gitops-token'"
                 }
            }
        }

    }

    post {
        success {
            slackSend(
                color: '#36a64f',
                message: "Job '${env.JOB_NAME}' (Build #${env.BUILD_NUMBER}) succeeded!",
                channel: '#notifications'
            )
        }
        failure {
            slackSend(
                color: '#ff0000',
                message: "Job '${env.JOB_NAME}' (Build #${env.BUILD_NUMBER}) failed!",
                channel: '#notifications'
            )
        }
    }


}