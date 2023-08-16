pipeline{
    // agent{
    //     label "jenkins-agent"
    // }

    agent any

    environment {
        APP_NAME = "lfc-training-testapi"
        RELEASE = "1.0.0"

        DOCKER_USER = "luciferifanum"
        DOCKER_PASS = "DOCKER"

        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        IMAGE_PATH = "${IMAGE_NAME}" + ":" + "${IMAGE_TAG}"

        SONAR_TOKEN = credentials("jenkins-sonarquebe-token")

        GIT_REPO_NAME = "https://github.com/luciferifanum/LFC.Deployments"
        GIT_USER_NAME = "luciferifanum"

        // JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")

    }

    stages{
        stage("Cleanup Workspace"){
            steps {
                cleanWs()
            }

        }

        stage("Checkout from SCM"){
            steps {
                git branch: 'develop', credentialsId: 'Github', url: 'https://github.com/luciferifanum/LFC.TestAPI'
            }
        }

        stage('Build Application') {
            steps {
                sh 'dotnet restore'
                // sh 'dotnet tool restore --configfile "src/LFC.Training.TestAPI/.config/dotnet-tool.json"'
                sh 'dotnet publish -c Release -o out -r linux-x64 --self-contained'
            }
        }

        stage("Build Docker Image") {
            steps {
                script {
                    docker_image = docker.build ("${IMAGE_NAME}")
                }
            }
        }

        stage("Push Docker Image") {
            steps {
                    script {
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image.push('latest')
                        docker_image.push("${IMAGE_TAG}")
                    }
                }
            }
        }

        stage("Trivy Scan") {
            steps {
                script {
                    sh '(docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image ${IMAGE_PATH} --no-progress --scanners vuln --exit-code 0 --severity HIGH,CRITICAL --format table)'
                }
            }
        }
    

        stage('Checkout ArgoCD K8S Manifest'){
            steps {
                git branch: 'main', credentialsId: 'Github', url: GIT_REPO_NAME
            }
        }

        // stage('git push') {
        //     steps {
        //         withCredentials([gitUsernamePassword(credentialsId: 'Github', gitToolName: 'Default')]) {
        //             sh '''
        //              # modify some files
        //             git add .
        //             git commit -m "register work"
        //             git push
        //             '''
        //         }
        //     }
        // }

        stage('Update K8S Manifest & Push to Repo'){
            steps {
                script  {
                    // withCredentials(credentialsId: 'Github') {
                        sh 'cat lfc-training-testapi-api/deployment.yaml'
                        sh 'sed -i "s|image: luciferifanum/lfc-training-testapi:[^ ]*|image: luciferifanum/lfc-training-testapi:${IMAGE_TAG}|g" lfc-training-testapi-api/deployment.yaml'
                        sh 'cat lfc-training-testapi-api/deployment.yaml'
                        sh 'git config user.email "dev@setenova.com"'
                        sh 'git config user.name "Setenova Dev Team"'
                        //sh 'git add lfc-training-testapi-api/deployment.yaml'
                        sh 'git add .'
                        sh 'git commit -m "Updated the deployment.yaml | Jenkins Pipeline"'
                        sh 'git status'
                        sh 'git remote -v'
                        //sh 'git remote set-url origin git@github.com:luciferifanum/LFC.Deployments.git'
                        sh 'git push origin HEAD:main'
                    // }
                }
            }
        }
        
        stage('Cleanup Artifacts') {
            steps {
                script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
                }
            }
        }

        // stage("Sonarqube Analysis") {
        //     steps {
        //         script {
        //             withSonarQubeEnv('sonarqube-server') {
        //                 sh 'dotnet-sonarscanner begin /k:"Test" /d:sonar.host.url="http://10.10.10.7:9000" /d:sonar.login="${SONAR_TOKEN}"'
        //                 sh 'dotnet build'
        //                 sh 'dotnet-sonarscanner end'
        //             }
        //         }
        //     }
        // }

        // stage("Quality Gate") {
        //     steps {
        //         script {
        //             waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
        //         }
        //     }
        // }
    }
}