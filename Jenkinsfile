def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
    'UNSTABLE': 'warning',
    'ABORTED': '#AAAAAA'
]

pipeline {
    agent any

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        TMDB_V3_API_KEY = credentials('tmdb-api-key')
        IMAGE_NAME = "ankii1212/netflix"
        CONTAINER_NAME1 = "netflix1"
        CONTAINER_NAME2 = "netflix2"
        CONTAINER_NAME3 = "netflix3"
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout from Git') {
            steps {
                git credentialsId: 'ssh-key', branch: 'feature', url: 'git@github.com:Ankitha-aa1/DevSecOps-Project.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                        sh '''
                            $SCANNER_HOME/bin/sonar-scanner \
                            -Dsonar.projectName=DevSecOps-Project \
                            -Dsonar.projectKey=DevSecOps-Project \
                            -Dsonar.login=$SONAR_TOKEN
                        '''
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
                    sh '''
                        for cname in $CONTAINER_NAME1 $CONTAINER_NAME2 $CONTAINER_NAME3; do
                          if docker ps -a --format '{{.Names}}' | grep -q $cname; then
                            echo "Stopping and removing container: $cname"
                            docker stop $cname
                            docker rm $cname
                          else
                            echo "Container $cname does not exist."
                          fi
                        done

                        if docker images -q $IMAGE_NAME > /dev/null; then
                            echo "Removing image: $IMAGE_NAME"
                            docker rmi -f $IMAGE_NAME
                        else
                            echo "Image $IMAGE_NAME does not exist."
                        fi
                    '''
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh 'docker build --build-arg TMDB_V3_API_KEY=$TMDB_V3_API_KEY -t $IMAGE_NAME .'
                        sh 'docker push $IMAGE_NAME'
                    }
                }
            }
        }

        stage('TRIVY Image Scan') {
            steps {
                sh "trivy image $IMAGE_NAME > trivyimage.txt"
            }
        }

        stage('Deploy to Containers') {
            steps {
                sh 'docker run -itd --name $CONTAINER_NAME1 -p 8082:80 $IMAGE_NAME'
                sh 'docker run -itd --name $CONTAINER_NAME2 -p 8083:80 $IMAGE_NAME'
                sh 'docker run -itd --name $CONTAINER_NAME3 -p 8084:80 $IMAGE_NAME'
            }
        }
    }

    post {
        failure {
            emailext(
                attachLog: true,
                subject: "'${currentBuild.result}' - Build Failed",
                body: """
                    <p><b>Project:</b> ${env.JOB_NAME}</p>
                    <p><b>Build Number:</b> ${env.BUILD_NUMBER}</p>
                    <p><b>URL:</b> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                """,
                to: 'mankitha91@gmail.com',
                attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
            )
        }

        always {
            script {
                echo 'Sending Slack notification...'
                slackSend(
                    channel: '#JENKINS NOTIFIER',
                    color: COLOR_MAP[currentBuild.currentResult] ?: '#AAAAAA',
                    message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\nMore info at: ${env.BUILD_URL}"
                )
            }
        }
    }
}
