pipeline {
    agent any
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        TMDB_V3_API_KEY = credentials('tmdb-api-key')
        IMAGE_NAME = "ankii1212/netflix"
        CONTAINER_NAME = "netflix"
    }
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Setup known_hosts') {
            steps {
                sh '''
                    mkdir -p ~/.ssh
                    ssh-keyscan github.com >> ~/.ssh/known_hosts
                    chmod 644 ~/.ssh/known_hosts
                '''
            }
        }

        stage('Checkout from Git') {
            steps {
                sshagent(['github-ssh-key']) {
                    git branch: 'feature', url: 'git@github.com:Ankitha-aa1/DevSecOps-Project.git'
                }
            }
        }

        stage("SonarQube Analysis") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh """
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=DevSecOps-Project \
                        -Dsonar.projectKey=DevSecOps-Project
                    """
                }
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                withCredentials([string(credentialsId: 'nvd-api-key', variable: 'NVD_API_KEY')]) {
                    dependencyCheck additionalArguments: "--scan ./ --disableYarnAudit --disableNodeAudit --nvdApiKey ${NVD_API_KEY}", odcInstallation: 'DP-Check'
                }
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('Trivy Filesystem Scan') {
            steps {
                sh "trivy fs . | tee trivyfs.txt"
            }
        }

        stage('Clean Up Docker Resources') {
            steps {
                script {
                    sh """
                        if docker ps -a --format '{{.Names}}' | grep -q '^${CONTAINER_NAME}\$'; then
                            echo "Stopping and removing container: ${CONTAINER_NAME}"
                            docker rm -f ${CONTAINER_NAME}
                        else
                            echo "Container ${CONTAINER_NAME} does not exist."
                        fi

                        if docker images -q ${IMAGE_NAME}; then
                            echo "Removing image: ${IMAGE_NAME}"
                            docker rmi -f ${IMAGE_NAME}
                        else
                            echo "Image ${IMAGE_NAME} does not exist."
                        fi
                    """
                }
            }
        }

        stage("Docker Build & Push") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker build --build-arg TMDB_V3_API_KEY=${TMDB_V3_API_KEY} -t ${IMAGE_NAME} ."
                        sh "docker push ${IMAGE_NAME}"
                    }
                }
            }
        }

        stage("Trivy Image Scan") {
            steps {
                sh "trivy image ${IMAGE_NAME} | tee trivyimage.txt"
            }
        }

        stage('Deploy to Container') {
            steps {
                sh "docker run -d --name ${CONTAINER_NAME} -p 8081:80 ${IMAGE_NAME}"
            }
        }
    }

    post {
        always {
            script {
                def colorMap = [
                    SUCCESS: '#2ECC71', // Green
                    FAILURE: '#E74C3C', // Red
                    UNSTABLE: '#F39C12', // Orange
                    ABORTED: '#95A5A6'  // Gray
                ]
                def buildStatus = currentBuild.currentResult
                def color = colorMap.get(buildStatus, '#3498DB') // Default: Blue
                emailext(
                    subject: "Build ${buildStatus}: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                    body: """<div style='font-family:Arial;'>
                        <h2 style='color:${color};'>Build ${buildStatus}</h2>
                        <p><b>Project:</b> ${env.JOB_NAME}</p>
                        <p><b>Build Number:</b> ${env.BUILD_NUMBER}</p>
                        <p><b>URL:</b> <a href='${env.BUILD_URL}'>Click here</a></p>
                    </div>""",
                    to: 'mankitha91@gmail.com',
                    mimeType: 'text/html',
                    attachLog: true,
                    attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
                )
            }
        }
    }
}
