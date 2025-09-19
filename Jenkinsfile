pipeline {
    agent any
    environment {
        //parameterizing the environment variables
        DOCKER_TAG = "${BUILD_NUMBER}"
        IMAGE_NAME = "rashmimurthi/fullstack"
        AWS_REGION = "us-east-1"
        SONAR_URL = "http://44.222.76.174:9000/" // SonarQube URL 
        GIT_REPO_NAME = "complete-cicd-project"
        GIT_USER_NAME ="rashmi-murthi"
        
    }
    tools {
        jdk 'java-17'
        maven 'maven'
    }
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/rashmi-murthi/complete-cicd-project.git'
            }
        }
        stage('Compile'){
            steps {
                sh "mvn compile"
            }
        }
        stage('Build'){
            steps {
                sh "mvn clean install"
            }
        }
        stage('Static Code Analysis(SonarQube)'){
            steps {
                withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]){
                sh "mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}"
                }
            }
        }
        stage('Docker Build and Tag the image'){
            steps {
                sh "docker build -t ${IMAGE_NAME}:${DOCKER_TAG} ."
            }
        }
        stage('Docker login and pushing image to dockerhub'){
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerHub', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USER')]){
                sh "echo $DOCKER_PASSWORD | docker login -u $DOCKER_USER --password-stdin"
                sh "docker push ${IMAGE_NAME}:${DOCKER_TAG}"
                }
            }
        }
        stage('Update the deployment file'){
            steps {
                withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        git config --global user.name "${GIT_USER_NAME}"
                        git config --global user.email "rashmi.pmurthi20@gmail.com"
                        sed -i "s|image: rashmimurthi/fullstack:[^ ]*|image: rashmimurthi/fullstack:${DOCKER_TAG}|g" argocd-manifest/deployment.yml
                        # Commit and push the updated deployment file
                        git add argocd-manifest/deployment.yml
                        git commit -m "Update deployment image to version ${DOCKER_TAG}"
                        git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main 
                    ''' 
                }
            }
        }
    }
    post {
        always {
            script {
                def jobName = env.JOB_NAME
                def buildNumber = env.BUILD_NUMBER
                def pipelineStatus = currentBuild.result ?: 'UNKNOWN'
                def bannerColor = pipelineStatus.toUpperCase() == 'SUCCESS' ? 'green' : 'red'

                def body = """
                    <html>
                    <body>
                    <div style="border: 4px solid ${bannerColor}; padding: 10px;">
                    <h2>${jobName} - Build ${buildNumber}</h2>
                    <div style="background-color: ${bannerColor}; padding: 10px;">
                    <h3 style="color: white;">Pipeline Status: ${pipelineStatus.toUpperCase()}</h3>
                    </div>
                    <p>Check the <a href="${BUILD_URL}">console output</a>.</p>
                    </div>
                    </body>
                    </html>
                """

                emailext (
                    subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus.toUpperCase()}",
                    body: body,
                    to: 'udayk0391@gmail.com',
                    from: 'rashmi.pmurthi20@gmail.com',
                    replyTo: 'rashmi.pmurthi20@gmail.com',
                    mimeType: 'text/html',
                    attachmentsPattern: '**/target/sonar-report.html'  // Path to the SonarQube report file
                )
            }
        }
    }
}

