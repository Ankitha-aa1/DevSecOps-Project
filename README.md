Deploy Netflix Clone on Cloud using Jenkins - DevSecOps Project

Phase 1: Initial Setup and Deployment
Step 1: Launch EC2 (Ubuntu 22.04): 
    . Provision EC2 instancewith instance type [t2.large] and volume [15GB]

    .  Connect to the instance using SSH.

Step 2: Clone the Code:
 . Update all the packages and then clone the code.

 . Clone your application's code repository onto the EC2 instance:

   git clone https://github.com/Sushmaa123/DevSecOps-Project.git

Step 3: Install Docker and Run the App Using a Container:
 . Set up Docker on the EC2 instance:
   
     sudo apt-get update
     sudo apt-get install docker.io -y
     sudo usermod -aG docker $USER  # Replace with your system's username, e.g., 'ubuntu'
     newgrp docker
     sudo chmod 777 /var/run/docker.sock

 . Build and run your application using Docker containers:

    docker build -t netflix .
    docker run -d --name netflix -p 8081:80 netflix:latest

    #to delete
    docker stop <containerid>
    docker rmi -f netflix

It will show an error cause you need API key

Step 4: Get the API Key:

. Open a web browser and navigate to TMDB (The Movie Database) website.

. Click on "Login" and create an account.

. Once logged in, go to your profile and select "Settings."

. Click on "API" from the left-side panel.

. Create a new API key by clicking "Create" and accepting the terms and conditions.

. Provide the required basic details and click "Submit."

. You will receive your TMDB API key.

Now recreate the Docker image with your api key:

docker build --build-arg TMDB_V3_API_KEY=<your-api-key> -t netflix .