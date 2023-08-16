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

        stage("Build Docker Image") {
            steps {
                script {
                    docker_image = docker.build ("${IMAGE_NAME}")
                }
            }
        }

        stage("Push Docker Image") {
            steps {
                echo IMAGE_PATH
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
                git branch: 'main', credentialsId: 'Github', url: 'https://github.com/luciferifanum/LFC.Deployments'
                // git credentialsId: '53ad6e8d-f843-40d1-8fb6-52ebd9a7504b', 
                // url: 'https://github.com/prabinav/argocd-my-app',
                // branch: 'main'
            }
        }

        // stage('Update K8S Manifest & Push to Repo'){
        //     steps {
        //         script {
        //             withCredentials([sshUserPrivateKey(credentialsId: '07b60c02-3cf2-4632-a791-32c9eb56aa38', keyFileVariable: 'SSH_KEY_FILE', passphraseVariable: 'SSH_PASSPHRASE', usernameVariable: 'SSH_USERNAME')]) {
        //                 sh '''
        //                 cat micro-app/microservice.yaml
        //                 sed -i "s|image: docker.io/iamprabin/cicd:[^ ]*|image: docker.io/iamprabin/cicd:${BUILD_NUMBER}|g" micro-app/microservice.yaml
        //                 cat micro-app/microservice.yaml
        //                 git add micro-app/microservice.yaml
        //                 git commit -m 'Updated the microservice.yaml | Jenkins Pipeline'
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
    }
}