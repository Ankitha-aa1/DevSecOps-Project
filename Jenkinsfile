def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger'
    ]
pipeline{
    agent any
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
        TMDB_V3_API_KEY = credentials('tmdb-api-key')
        IMAGE_NAME = "ankii1212/netflix" // Name of the image created in Jenkins
        CONTAINER_NAME1 = "netflix1" // Name of the container created in Jenkins
        CONTAINER_NAME2 = "netflix2"
        CONTAINER_NAME3 = "netflix3"
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'feature', url: 'git@github.com:Ankitha-aa1/DevSecOps-Project.git'
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
        stage('Clean Up Docker Resources') {
            steps {
                script {
                    // Remove the specific container
                    sh '''
                    if docker ps -a --format '{{.Names}}' | grep -q $CONTAINER_NAME; then
                        echo "Stopping and removing container: $CONTAINER_NAME"
                        docker stop $CONTAINER_NAME
                        docker rm $CONTAINER_NAME
                    else
                        echo "Container $CONTAINER_NAME does not exist."
                    fi
                    '''

                    // Remove the specific image
                    sh '''
                    if docker images -q $IMAGE_NAME; then
                        echo "Removing image: $IMAGE_NAME"
                        docker rmi -f $IMAGE_NAME
                    else
                        echo "Image $IMAGE_NAME does not exist."
                    fi
                    '''
                }
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker-cred'){   
                       sh 'docker build --build-arg TMDB_V3_API_KEY=$TMDB_V3_API_KEY -t $IMAGE_NAME .'
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
                sh 'docker run -itd --name $CONTAINER_NAME1 -p 8082:80 $IMAGE_NAME'
                sh 'docker run -itd --name $CONTAINER_NAME2 -p 8083:80 $IMAGE_NAME'
                sh 'docker run -itd --name $CONTAINER_NAME3 -p 8084:80 $IMAGE_NAME'
            }
        }
    }
post {
     failure {
        emailext attachLog: true,
            subject: "'${currentBuild.result}'",
            body: "Project: ${env.JOB_NAME}<br/>" +
                "Build Number: ${env.BUILD_NUMBER}<br/>" +
                "URL: ${env.BUILD_URL}<br/>",
            to: 'mankitha91@gmail.com',                               
            attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
        }

        always {
            echo 'slack Notification.'
            slackSend channel: '#all-netflix',
            color: COLOR_MAP [currentBuild.currentResult],
            message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URl}"
            
        }
    }
}