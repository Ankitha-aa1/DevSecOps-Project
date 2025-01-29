**Deploy Netflix Clone on Cloud using Jenkins - DevSecOps Project**

**Phase 1: Initial Setup and Deployment**

Step 1: Launch EC2 (Ubuntu 22.04):
    - Provision EC2 instancewith instance type [t2.large] and volume [15GB]
    - Connect to the instance using SSH.

Step 2: Clone the Code:
 - Update all the packages and then clone the code.
 - Clone your application's code repository onto the EC2 instance:
   git clone https://github.com/Sushmaa123/DevSecOps-Project.git

Step 3: Install Docker and Run the App Using a Container:
 - Set up Docker on the EC2 instance:

    sudo apt-get update

    sudo apt-get install docker.io -y

    sudo usermod -aG docker $USER  # Replace with your system's username, e.g., 'ubuntu'

    newgrp docker

    sudo chmod 777 /var/run/docker.sock

 - Build and run your application using Docker containers:

     docker build -t netflix .

     docker run -d --name netflix -p 8081:80 netflix:latest

     #to delete

     docker stop <containerid>

     docker rmi -f netflix

It will show an error cause you need API key

Step 4: Get the API Key:

- Open a web browser and navigate to TMDB (The Movie Database) website.

- Click on "Login" and create an account.

- Once logged in, go to your profile and select "Settings."

- Click on "API" from the left-side panel.

- Create a new API key by clicking "Create" and accepting the terms and conditions.

- Provide the required basic details and click "Submit."

- You will receive your TMDB API key.

Now recreate the Docker image with your api key:

docker build --build-arg TMDB_V3_API_KEY=<your-api-key> -t netflix .

- Access the application and check:

  http://<ip-address>:8081

**Install Sonarqube and Trivy**

Install SonarQube and Trivy on the EC2 instance to scan for vulnerabilities.

Install sonarqube:

sudo docker run -itd --name sonarqube -p 9000:9000 sonarqube

Access Sonarqube using public-ip:9000  (by default username & password is admin)

Install Trivy:

sudo apt-get install wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy        

to scan image using trivy:

trivy image <imageid>

**Phase2:CI/CD Setup to run Netflix using Jenkins**

1.Install Jenkins for Automation
- Install Jenkins on the EC2 instance to automate deployment
    - install Java
sudo apt update
sudo apt install openjdk-17-jdk -y
java -version

#jenkins
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins -y
sudo service jenkins start
sudo service jenkins status

- Access Jenkins in a web browser using the public IP of your EC2 instance.
  
  public-ip:8080

- Copy the path for administrator password
- Run the below cmd on the instance to get the password.
  cat /var/jenkins_home/secrets/initialAdminPassword
- Copy the password and paste it on jenkins
- Then install suggested plugins
- Create a User by adding name,password and email

2. Install necessary plugins in jenkins:

Goto Manage Jenkins →Plugins → Available Plugins →

Install below plugins: 

1. Sonarqube scanner

2. OWASP dependency check

3. NodeJS Plugin

4. Docker, docker pipeline, docker build-step, cloudbees docker build&publish

5. Slack Notification

6. pipeline stage view

**Configure Java and Nodejs in Global Tool Configuration**

Goto Manage Jenkins → Tools → Install JDK(17) and NodeJs(16)→ Click on Apply and Save
