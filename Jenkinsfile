pipeline{
    agent any
    
    environment {
        dotnet ='/usr/bin/dotnet'
        APP_NAME = 'eshopwebmvc'
        PROJECT_ID = 'mygcpproject-82449'
        CLUSTER_NAME = 'cluster-1'
        LOCATION = 'asia-south1-c'
        CREDENTIALS_ID = 'MyGcpProject 82449'
        }
        
    triggers {
        pollSCM 'H * * * *'
    }
    stages{
        stage ('Clean workspace') {
            steps {
                cleanWs()
            }
        }        
        stage('Checkout') {
            steps {
                checkout scm
                }
        }
        stage('Build Docker build') {
            steps{
                withCredentials([usernamePassword(credentialsId: 'NugetCredentials', passwordVariable: 'Nuget_CustomFeedPassword', usernameVariable: 'Nuget_CustomFeedUserName')]) {
                    // sh "docker build --pull -t eshopwebmvc -f src/Web/Dockerfile --build-arg Nuget_CustomFeedUserName --build-arg Nuget_CustomFeedPassword . --no-cache --progress=plain"
                    script {
                        myapp = docker.build("gtaroadrash/${env.APP_NAME}:${env.BUILD_ID}", "--build-arg Nuget_CustomFeedUserName", "--build-arg Nuget_CustomFeedPassword", "-f src/Web/Dockerfile .")
                    // docker-compose build --build-arg Nuget_CustomFeedUserName --build-arg Nuget_CustomFeedPassword
                    }
                }
            }
        }
        stage('Push image build') {
            steps{
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                            myapp.push("latest")
                            myapp.push("${env.BUILD_ID}")
                    }
                }
            }
        }
        stage('Deploy to GKE') {
            steps{
                sh "sed -i 's/eshopwebmvc:latest/${env.APP_NAME}:${env.BUILD_ID}/g' src/Web/deployment.yaml"
                step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, manifestPattern: 'src/Web/deployment.yaml', credentialsId: env.CREDENTIALS_ID, verifyDeployments: true])
            }
        }        
    }
    post{
        always{
            emailext body: "${currentBuild.currentResult}: Job   ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}",
            recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']],
            subject: "Jenkins Build ${currentBuild.currentResult}: Job ${env.JOB_NAME}"
        }
    }    
 }
