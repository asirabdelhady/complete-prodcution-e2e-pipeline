pipeline{
    agent{
        label "jenkins-agent"
    }
    tools{
        jdk 'Java17'
        maven 'Maven3'
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
        

    }

}