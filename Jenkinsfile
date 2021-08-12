pipeline{
    agent any
    
    environment {
        dotnet ='/usr/bin/dotnet'
        APP_NAME = 'eshopwebmvc'
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
                    myapp = docker.build("gtaroadrash/${env.APP_NAME}:${env.BUILD_ID}")
                    // docker-compose build --build-arg Nuget_CustomFeedUserName --build-arg Nuget_CustomFeedPassword
                    }
                }
        }
        stage('Push image build') {
            steps{
                docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                    myapp.push("latest")
                    myapp.push("${env.BUILD_ID}")
                }
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