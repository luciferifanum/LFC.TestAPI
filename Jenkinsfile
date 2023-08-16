pipeline{
    // agent{
    //     label "jenkins-agent"
    // }

    agent any

    // tools {
    //     jdk 'Java17'
    //     maven 'Maven3'
    // }

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

        stage("Sonarqube Analysis") {
            steps {
                script {
                    withSonarQubeEnv('sonarqube-server') {
                        sh 'dotnet-sonarscanner begin /k:"Test" /d:sonar.host.url="http://10.10.10.7:9000" /d:sonar.login="${SONAR_TOKEN}"'
                        sh 'dotnet build'
                        sh 'dotnet-sonarscanner end'
                    }
                }
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
                    sh "docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image ${IMAGE_PATH} --no-progress --scanners vuln --exit-code 0 --severity HIGH,CRITICAL --format table"
                }
            }
        }

        stage ('Cleanup Artifacts') {
            steps {
                script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
                }
            }
        }

        // stage("Quality Gate") {
        //     steps {
        //         script {
        //             waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
        //         }
        //     }

        // }

        // stage("Trigger CD Pipeline") {
        //     steps {
        //         script {
        //             sh "curl -v -k --user admin:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'https://jenkins.dev.dman.cloud/job/gitops-complete-pipeline/buildWithParameters?token=gitops-token'"
        //         }
        //     }

        // }

    }

    // post {
    //     failure {
    //         emailext body: '''${SCRIPT, template="groovy-html.template"}''', 
    //                 subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - Failed", 
    //                 mimeType: 'text/html',to: "dmistry@yourhostdirect.com"
    //         }
    //      success {
    //            emailext body: '''${SCRIPT, template="groovy-html.template"}''', 
    //                 subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - Successful", 
    //                 mimeType: 'text/html',to: "dmistry@yourhostdirect.com"
    //       }      
    // }
}