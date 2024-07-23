pipeline{
    agent any
       
    tools {
        jdk "jdk17"
        nodejs "node"
    }
    environment {
        SONAR_HOME = tool "sonar-tool"
        }

    stages{
        stage("Clean workspace"){
            steps{
                cleanWs()
            }
        }
        stage("Check from git"){
            steps{
                git branch: 'main', url: 'https://github.com/haleem00/DevSecOps-Project.git'
            }
        }
        stage("Sonerqube"){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh '''
                        $SONAR_HOME/bin/sonar-tool -Dsonar.projectName=Netflix \
                        -Dsonar.projectKey=Netflix
                    '''
                    
                }
            }
        }
        stage("Quilty gate"){
            steps{
                waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
            }
        }
        // stage("Clean workspace"){
        //     steps{
        //         cleanWs()
        //     }
        // }
    }

}