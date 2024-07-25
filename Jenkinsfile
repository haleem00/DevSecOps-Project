pipeline {
    agent any

    tools {
        jdk "jdk17"
        nodejs "node"
    }

    environment {
        SONAR_HOME = tool "sonar-tool"
        DOCKERIMAGE = 'haleemo/netfilx'
        DOCKERPASS = "dockerhub"
        ARGOCD_SERVER = 'http://3.85.146.238:31096'
        ARGOCD_APPLICATION = 'netflix'
        ARGOCD_PROJECT = 'default'
    }

    stages {
        stage("Clean workspace") {
            steps {
                cleanWs()
            }
        }

        stage("Check from git") {
            steps {
                git branch: 'main', url: 'https://github.com/haleem00/DevSecOps-Project.git'
            }
        }

        stage("Install Dependencies") {
            steps {
                sh 'npm install'
            }
        }

        stage("OWASP FS SCAN") {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --format XML --out dependency-check-report.xml', odcInstallation: 'dependency-check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage("SonarQube") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                        $SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                        -Dsonar.projectKey=Netflix
                    '''
                }
            }
        }

        // stage("Quality Gate") {
        //     steps {
        //         waitForQualityGate abortPipeline: true, credentialsId: 'Sonar-token'
        //     }
        // }

        stage("TRIVY FS Scan") {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }

        stage("Build and Push Docker Image") {
            steps {
                script {
                    dockerImage = docker.build("${DOCKERIMAGE}:V${env.BUILD_NUMBER}")
                    withDockerRegistry([url: '', credentialsId: 'dockerhub']) {
                        dockerImage.push("V${env.BUILD_NUMBER}")
                        dockerImage.push('latest')
                        sh "trivy image haleemo/netfilx:latest > trivyimage.txt"
                        sh "docker rmi haleemo/netfilx:latest haleemo/netfilx:V${env.BUILD_NUMBER}"
                    }
                }
            }
        }

        // stage("Build and Push Docker Image") {
        //     steps {
        //         script {
        //             withDockerRegistry('', credentialsId: 'dockerhub') {
        //                 sh "docker build --build-arg TMDB_V3_API_KEY=b16d8eed9624150739d555b64b2a569c -t netflix ."
        //                 sh "docker tag netflix haleemo/netfilx:latest"
        //                 sh "docker tag netflix haleemo/netfilx:$BUILD_NUMBER"
        //                 sh "docker push haleemo/netfilx:latest"
        //                 sh "docker push haleemo/netfilx:$BUILD_NUMBER"
        //                 sh "docker rmi haleemo/netfilx:latest haleemo/netfilx:$BUILD_NUMBER"
        //             }
        //         }
        //     }
        // }
    }

    post {
        success {
            script {
                withCredentials([string(credentialsId: 'argo-token', variable: 'ARGOCD_TOKEN')]) {
                    def response = httpRequest(
                        url: "${ARGOCD_SERVER}/api/v1/applications/${ARGOCD_APPLICATION}/sync",
                        httpMode: 'POST',
                        customHeaders: [[name: 'Authorization', value: "Bearer ${ARGOCD_TOKEN}"]],
                        contentType: 'APPLICATION_JSON',
                        requestBody: '''{
                            "project": "${ARGOCD_PROJECT}"
                        }'''
                    )
                    echo "Argo CD Sync Response: ${response.content}"
                }
            }
        }
    }
}
