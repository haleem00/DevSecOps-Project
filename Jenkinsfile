pipeline{
    agent any
       
    tools {
        jdk "jdk17"
        nodejs "node"
    }
    environment {
        SONAR_HOME = tool "sonar-tool"
        DOCKERIMAGE = 'haleemo/netfilx'
        DOCKERPASS = "dockerhub"
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
        stage("Install Dependencies"){
            steps{
                sh 'npm install'
            }
        }
        stage("OWASP FS SCAN"){
            steps{
                dependencyCheck additionalArguments: '--scan ./ --format XML --out dependency-check-report.xml', odcInstallation: 'dependency-check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage("Sonerqube"){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh '''
                        $SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                        -Dsonar.projectKey=Netflix
                    '''
                    
                }
            }
        }
        // stage("Quilty gate"){
        //     steps{
        //         waitForQualityGate abortPipeline: true, credentialsId: 'Sonar-token'

        //     }
        // }
        stage("Build and push docker image"){
            steps{
                script {
                    dockerImage = docker.build("${DOCKERIMAGE}:V${env.BUILD_NUMBER}")
                    withDockerRegistry([url: '', credentialsId: 'dockerhub']) {
                        dockerImage.push("V${env.BUILD_NUMBER}")
                        dockerImage.push('latest')
                        sh "docker rmi haleemo/netfilx:latest haleemo/netfilx:${env.BUILD_NUMBER}"
                    }
                }
            }
        }
        // stage("Build and push docker image"){
        //     steps{
        //         script {
        //             withDockerRegistry('', credentialsId: 'dockerhub') {

        //             sh "docker build --build-arg TMDB_V3_API_KEY=b16d8eed9624150739d555b64b2a569c -t netflix ."
        //             sh "docker tag netflix haleemo/netfilx:latest"
        //             sh "docker tag netflix haleemo/netfilx:$BUILD_NUMBER"
        //             sh "docker push haleemo/netfilx:latest"
        //             sh "docker push haleemo/netfilx:$BUILD_NUMBER"
        //             sh "docker rmi haleemo/netfilx:latest haleemo/netfilx:$BUILD_NUMBER"
        //             }
        //         }
        //     }
        // }
    }

}