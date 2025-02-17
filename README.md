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


# Phase 4: Monitoring

## Install Prometheus and Grafana

Set up Prometheus and Grafana to monitor your application.

---

## Installing Prometheus

### 1. Create a dedicated Linux user for Prometheus and download Prometheus:
```sh
sudo useradd --system --no-create-home --shell /bin/false prometheus
wget https://github.com/prometheus/prometheus/releases/download/v2.47.1/prometheus-2.47.1.linux-amd64.tar.gz

## 2. Extract Prometheus files, move them, and create necessary directories:

tar -xvf prometheus-2.47.1.linux-amd64.tar.gz
cd prometheus-2.47.1.linux-amd64/
sudo mkdir -p /data /etc/prometheus
sudo mv prometheus promtool /usr/local/bin/
sudo mv consoles/ console_libraries/ /etc/prometheus/
sudo mv prometheus.yml /etc/prometheus/prometheus.yml

## Step 3: Set Ownership
sudo chown -R prometheus:prometheus /etc/prometheus/ /data/

## Step 4: Create a systemd Service File
sudo nano /etc/systemd/system/prometheus.service

Add the following content:
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

StartLimitIntervalSec=500
StartLimitBurst=5

[Service]
User=prometheus
Group=prometheus
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/data \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.listen-address=0.0.0.0:9090 \
  --web.enable-lifecycle

[Install]
WantedBy=multi-user.target

## Step 5: Enable and Start Prometheus
sudo systemctl enable prometheus
sudo systemctl start prometheus

## Step 6: Verify Prometheus Status
sudo systemctl status prometheus

## Access Prometheus via:
http://<your-server-ip>:9090


** Installing Node Exporter **
##Step 1: Create a Dedicated User and Download Node Exporter
sudo useradd --system --no-create-home --shell /bin/false node_exporter
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz

## Step 2: Extract and Move Files
tar -xvf node_exporter-1.6.1.linux-amd64.tar.gz
sudo mv node_exporter-1.6.1.linux-amd64/node_exporter /usr/local/bin/
rm -rf node_exporter*

## Step 3: Create a systemd Service File
sudo nano /etc/systemd/system/node_exporter.service


## Add the following content:
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

StartLimitIntervalSec=500
StartLimitBurst=5

[Service]
User=node_exporter
Group=node_exporter
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/node_exporter --collector.logind

[Install]
WantedBy=multi-user.target


**Step 4: Enable and Start Node Exporter **
sudo systemctl enable node_exporter
sudo systemctl start node_exporter

## Step 5: Verify Node Exporter Status
Step 5: Verify Node Exporter Status

** Configure Prometheus to Scrape Metrics**
Modify /etc/prometheus/prometheus.yml:

global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']

  - job_name: 'jenkins'
    metrics_path: '/prometheus'
    static_configs:
      - targets: ['<your-jenkins-ip>:<your-jenkins-port>']

## Validate Prometheus Configuration
promtool check config /etc/prometheus/prometheus.yml

##Reload Prometheus Configuration
curl -X POST http://localhost:9090/-/reload

##Access Prometheus targets:
http://<your-prometheus-ip>:9090/targets

**Installing Grafana**
## Step 1: Install Dependencies
sudo apt-get update
sudo apt-get install -y apt-transport-https software-properties-common

## Step 2: Add the GPG Key
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -

## Step 3: Add Grafana Repository
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list

## Step 4: Install Grafana
sudo apt-get update
sudo apt-get -y install grafana

## Step 5: Enable and Start Grafana
sudo systemctl enable grafana-server
sudo systemctl start grafana-server

## Step 6: Verify Grafana Status
sudo systemctl status grafana-server

## Step 7: Access Grafana
http://<your-server-ip>:3000

Default Credentials:

Username: admin
Password: admin (change upon first login)

## Step 8: Change the Default Password
Grafana will prompt you to set a new password upon first login.

** Adding Prometheus Data Source in Grafana **
1.Click on ⚙️ Configuration in the left sidebar.
2.Select Data Sources.
3.Click Add data source.
4.Choose Prometheus.
5.In the HTTP section:
6.Set URL to http://localhost:9090
7.Click Save & Test.

** Importing a Preconfigured Dashboard **
1.Click on + Create in the left sidebar.
2.Select Dashboard.
3.Click Import.
4.Enter the dashboard ID (e.g., 1860).
5.Click Load.
6.Select the Prometheus data source.
7.Click Import.

** Configure Prometheus Plugin Integration with Jenkins **
## To monitor Jenkins with Prometheus:

Install the Prometheus Metrics Plugin in Jenkins.
Configure Jenkins to expose metrics at /prometheus.
Update prometheus.yml as shown earlier.
Restart Prometheus and Jenkins.
