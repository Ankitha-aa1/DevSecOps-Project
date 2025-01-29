# Deploy Netflix Clone on Cloud using Jenkins - DevSecOps Project

## Phase 1: Initial Setup and Deployment

### Step 1: Launch EC2 (Ubuntu 22.04)
- Provision an EC2 instance with:
  - Instance type: `t2.large`
  - Volume: `15GB`
- Connect to the instance using SSH.

### Step 2: Clone the Code
- Update all packages and clone the repository:
  ```bash
  sudo apt-get update
  git clone https://github.com/Sushmaa123/DevSecOps-Project.git
  ```

### Step 3: Install Docker and Run the App Using a Container
- Install Docker:
  ```bash
  sudo apt-get update
  sudo apt-get install docker.io -y
  sudo usermod -aG docker $USER  # Replace with your system's username, e.g., 'ubuntu'
  newgrp docker
  sudo chmod 777 /var/run/docker.sock
  ```

- Build and run the application using Docker:
  ```bash
  docker build -t netflix .
  docker run -d --name netflix -p 8081:80 netflix:latest
  ```

- To stop and remove the container:
  ```bash
  docker stop <containerid>
  docker rmi -f netflix
  ```

**Note:** You need an API key for the application to work.

### Step 4: Get the API Key
- Open a web browser and navigate to [TMDB (The Movie Database)](https://www.themoviedb.org/).
- Log in or create an account.
- Navigate to `Settings` > `API`.
- Create a new API key and accept the terms.
- Use the API key when building the Docker image:
  ```bash
  docker build --build-arg TMDB_V3_API_KEY=<your-api-key> -t netflix .
  ```

- Access the application at:
  ```
  http://<ip-address>:8081
  ```

## Install SonarQube and Trivy
### Install SonarQube
```bash
sudo docker run -itd --name sonarqube -p 9000:9000 sonarqube
```
- Access SonarQube at: `http://<public-ip>:9000` (Default credentials: admin/admin)

### Install Trivy
```bash
sudo apt-get install wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy
```
- Scan an image using Trivy:
  ```bash
  trivy image <imageid>
  ```

## Phase 2: CI/CD Setup to Run Netflix Using Jenkins

### Step 1: Install Jenkins for Automation
- Install Java:
  ```bash
  sudo apt update
  sudo apt install openjdk-17-jdk -y
  java -version
  ```
- Install Jenkins:
  ```bash
  sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
    https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
  echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
    https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
    /etc/apt/sources.list.d/jenkins.list > /dev/null
  sudo apt-get update
  sudo apt-get install jenkins -y
  sudo service jenkins start
  sudo service jenkins status
  ```
- Access Jenkins in a web browser:
  ```
  http://<public-ip>:8080
  ```
- Retrieve the administrator password:
  ```bash
  cat /var/jenkins_home/secrets/initialAdminPassword
  ```
- Paste the password in Jenkins setup, install suggested plugins, and create a user.

### Step 2: Install Necessary Plugins in Jenkins
- Go to `Manage Jenkins` → `Plugins` → `Available Plugins`.
- Install the following plugins:
  1. SonarQube Scanner
  2. OWASP Dependency Check
  3. NodeJS Plugin
  4. Docker, Docker Pipeline, Docker Build-Step, CloudBees Docker Build & Publish
  5. Slack Notification
  6. Pipeline Stage View

### Step 3: Configure Java and Node.js in Global Tool Configuration

 Global Tool Configuration is used to configure different tools that we install using Plugins

- Go to `Manage Jenkins` → `Tools` → Install:
  - Nodejs16
  - Sonar-scanner
  - DP-Check
- Click `Apply` and `Save`.

### step 4: Configure Sonarqube and Slack in System Configuration

The Configure System option is used in Jenkins to configure different server

- Go to sonarqube server and create a token
  
  - go to `administrator` -> `security` -> `users` -> `token`

- Go to system configure in jenkins
  
  **Sonarqube**

   - select environment variables
   - Add name [sonar-server] and add credentials

  **Slack**

    - Create a workspace and add channel
    - Go to slack app and add jenkins CI to slack
    - Get the subdomain and credentialsID
    - add subdomain and credentials in jenkins
    - Click on apply and save

### step 5: Create a Pipeline Job

   - Go to dashboard of jenkins
   - click on new item and give name for the job then select pipeline job
   - Create jenkins webhook

      - in the build triggers select githubhook trigger for scm
      - then go to your github repository, open settings and select webhook
      - add payload url then select application/json in content type and save it





