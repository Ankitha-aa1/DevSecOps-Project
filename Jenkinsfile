def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger'
]

pipeline {
    agent any

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        TMDB_V3_API_KEY = credentials('tmdb-api-key')
        IMAGE_NAME = "ankii1212/netflix"
        CONTAINERS = 'container1,container2,container3'
        PORTS = '8082,8083,8084'
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout from Git') {
            steps {
                git branch: 'feature', credentialsId: 'github-token', url: 'https://github.com/Ankitha-aa1/DevSecOps-Project.git'
            }
        }

        stage('Sonarqube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                        sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=DevSecOps-Project \
                        -Dsonar.projectKey=DevSecOps-Project'''
                    }
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
                    def containerList = env.CONTAINERS.split(',')

                    for (c in containerList) {
                        sh """
                        if docker ps -a --format '{{.Names}}' | grep -q ^${c}\$; then
                            echo "Stopping and removing container: ${c}"
                            docker stop ${c}
                            docker rm ${c}
                        else
                            echo "Container ${c} does not exist."
                        fi
                        """
                    }

                    sh """
                    if docker images -q $IMAGE_NAME; then
                        echo "Removing image: $IMAGE_NAME"
                        docker rmi -f $IMAGE_NAME
                    else
                        echo "Image $IMAGE_NAME does not exist."
                    fi
                    """
                }
            }
        }

        stage("Docker Build & Push") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh 'docker build --build-arg TMDB_V3_API_KEY=$TMDB_V3_API_KEY -t $IMAGE_NAME .'
                        sh 'docker push $IMAGE_NAME'
                    }
                }
            }
        }

        stage("TRIVY IMAGE SCAN") {
            steps {
                sh "trivy image $IMAGE_NAME > trivyimage.txt"
            }
        }

        stage('Deploy to container') {
            steps {
                script {
                    def containerList = env.CONTAINERS.split(',')
                    def portList = env.PORTS.split(',')

                    for (int i = 0; i < containerList.size(); i++) {
                        def container = containerList[i]
                        def port = portList[i]
                        sh "docker run -d --name ${container} -p ${port}:80 ${IMAGE_NAME}"
                    }
                }
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
            echo 'Slack Notification.'
            slackSend channel: '#all-netflix',
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \nMore info at: ${env.BUILD_URL}"
        }
    }
}
