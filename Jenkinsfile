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

        GIT_CD_REPO = "https://github.com/luciferifanum/LFC.Deployments"
        GIT_USER_NAME = "****************"

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
                git branch: 'main', credentialsId: 'Github', url: GIT_CD_REPO
            }
        }

        stage('Update K8S Manifest & Push to Repo'){
            steps {
                script  {
                    sh 'cat lfc-training-testapi-api/deployment.yaml'
                    sh 'sed -i "s|image: luciferifanum/lfc-training-testapi:[^ ]*|image: luciferifanum/lfc-training-testapi:${IMAGE_TAG}|g" lfc-training-testapi-api/deployment.yaml'
                    sh 'cat lfc-training-testapi-api/deployment.yaml'
                    sh 'git config  user.email "dev@setenova.com'
                    sh 'git config  user.name "Setenova Dev Team"'
                    sh 'git add lfc-training-testapi-api/deployment.yaml'
                    sh 'git commit -m "Updated the service.yaml | Jenkins Pipeline"'
                    sh 'git remote -v'
                }
            }
        }
        // stage ('Updating the Deployment File') {
        //     steps {
        //         withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]){
        //             sh '''
                    
        //                 git pull https://github.com/****************/CI-CD-PIPELINE.git
        //                 git config  user.email "****************.com"
        //                 git config  user.name "****************"
        //                 BUILD_NUMBER=${BUILD_NUMBER}
        //                 sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" ArgoCD/deployments.yml
        //                 git add ArgoCD/deployments.yml
        //                 git commit -m "updated the image ${BUILD_NUMBER}"
        //                 git push @github.com/${GIT_USER_NAME}/${GIT_REPO_NAME">@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME">@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME">https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
        //             '''
        //         }
        //     }
        // }

        // stage('Update K8S Manifest & Push to Repo'){
        //     steps {
        //         script {
        //             withCredentials([sshUserPrivateKey(credentialsId: '07b60c02-3cf2-4632-a791-32c9eb56aa38', keyFileVariable: 'SSH_KEY_FILE', passphraseVariable: 'SSH_PASSPHRASE', usernameVariable: 'SSH_USERNAME')]) {
        //                 sh '''
        //                 cat lfc-training-testapi-api/service.yaml
        //                 sed -i "s|image: docker.io/iamprabin/cicd:[^ ]*|image: docker.io/iamprabin/cicd:${BUILD_NUMBER}|g" micro-app/microservice.yaml
        //                 cat lfc-training-testapi-api/service.yaml
        //                 git add lfc-training-testapi-api/service.yaml
        //                 git commit -m 'Updated the service.yaml | Jenkins Pipeline'
        //                 git remote -v
     
        //                 # Set the remote URL to use SSH
        //                 git remote set-url origin git@github.com:prabinav/argocd-my-app.git
        
        //                 # Use ssh-agent to add the SSH key and push the changes
        //                 ssh-agent bash -c 'ssh-add ${SSH_KEY_FILE}; git push origin HEAD:main'
        //                 '''
        //             }
        //         }
        //     }    
        // }
        
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