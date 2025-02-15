pipeline{
    agent any
    tools{
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
        NVD_API_KEY = '920cd1a1-707d-4d34-bebb-338f0ff83fe5'
        IMAGE_NAME = "sushmaagowdaa/netflix" // Name of the image created in Jenkins
        CONTAINER_NAME = "netflix" // Name of the container created in Jenkins
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git 'https://github.com/Sushmaa123/DevSecOps-Project.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=DevSecOps-Project \
                    -Dsonar.projectKey=DevSecOps-Project'''
                }
            }
        }
       
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('OWASP FS SCAN') {
          steps {
        withCredentials([string(credentialsId: 'nvd-api-key', variable: 'NVD_API_KEY')]) {
             dependencyCheck additionalArguments: "--scan ./ --disableYarnAudit --disableNodeAudit --nvdApiKey ${NVD_API_KEY}", odcInstallation: 'DP-Check'
             }
             dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
          }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"     
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker-cred'){   
                       sh 'docker build --build-arg TMDB_V3_API_KEY=4434ab883e16321587b56db6effdfa6c -t $IMAGE_NAME'
                       sh 'docker push $IMAGE_NAME'
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image $IMAGE_NAME > trivyimage.txt"
            }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -itd --name $CONTAINER_NAME -p 8081:80 $IMAGE_NAME'
            }
        }
    }
post {
     always {
        emailext attachLog: true,
            subject: "'${currentBuild.result}'",
            body: "Project: ${env.JOB_NAME}<br/>" +
                "Build Number: ${env.BUILD_NUMBER}<br/>" +
                "URL: ${env.BUILD_URL}<br/>",
            to: 'sushmaananda999@gmail.com',                               
            attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
        }
    }
}

