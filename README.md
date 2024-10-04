# AWS-Containerization

## Introduction

The objective of this assignment is to familiarize yourself with Docker and containerization by Dockerizing a simple HTML page using Nginx as the web server.

## Prerequisites

- Basic understanding of Docker (images, containers, Dockerfiles)
- Familiarity with Nginx configuration for static files
- Docker installed and running on AWS EC2
- Simple HTML file ready
- Basic command-line knowledge
- Text editor for writing Dockerfile (e.g., VS Code)
- AWS ECR
- AWS EC2

## Installation

1. **Setting Up Docker on AWS EC2 instance**:
   - Update Package Index:
     ```bash
     sudo apt-get update
     ```
   - Install Docker:
     ```bash
     sudo apt install docker.io -y
     ```
   - Start Nginx Service:
     ```bash
     sudo systemctl start docker
     ```
   - Enable Docker:
     ```bash
     sudo systemctl enable docker
     ```

## Development Phase

1. **Create Sample HTML Website**:
   - A sample HTML website was created in VS code and pushed to GitHub using Git commands in Git Bash.

2. **Developed a Dockerfile in VS code**:
   
   - A dockerfile was written with the aim to run the static HTML page on nginx exposed via port 80.
     
     ```bash
     # Use the official Nginx base image
     FROM nginx:alpine

     # Copy custom Nginx configuration
     COPY nginx.conf /etc/nginx/conf.d/default.conf

     # Copy the HTML file to the appropriate directory
     COPY index.html /usr/share/nginx/html/index.html

     # Expose port 80
     EXPOSE 80

     # Start the Nginx server
     CMD ["nginx", "-g", "daemon off;"]
     ```

- **Key Components:**
  - **Specifies the base image:**
    - **Example:** `FROM nginx:alpine`
    - **Purpose:** Sets the foundation for the Docker image, in this case using Nginx with Alpine Linux.

  - **COPY or ADD Instruction:**
    - **Example:** `COPY ./index.html /usr/share/nginx/html/index.html`
    - **Purpose:** Copies files from the host machine to the Docker image, such as adding the `index.html` file to Nginx’s directory for serving content.

  - **EXPOSE Instruction:**
    - **Example:** `EXPOSE 80`
    - **Purpose:** Defines the port the container will listen on, making port 80 available for HTTP traffic to allow Nginx to serve requests.

  - **CMD or ENTRYPOINT Instruction:**
    - **Example:** `CMD ["nginx", "-g", "daemon off;"]`
    - **Purpose:** Defines the default command that the container will run when it starts, ensuring Nginx runs in the foreground, keeping the container alive.

3. **Creating a nginx.conf file**:
   
   - Developed a nginx to conf file which will define two things, one the port on which the nginx should run and second the root location where the static HMTL file shall be placed.
     
     ```bash
     server {
         listen 80;
         server_name localhost;

         location / {
             root /usr/share/nginx/html;
             index index.html;
         }
     }

     ```
 - **Key Components:**
   - **Server Block:**
     - `listen 80;`: Specifies that the server listens on port 80 (default HTTP port).
     - `server_name localhost;`: The server name is set to "localhost", meaning it will respond to requests sent to this address.

   - **Location Block:**
     - `location / { ... }`: Defines the root location where requests will be served.
     - `root /usr/share/nginx/html;`: Sets the root directory for serving files, in this case, it is `/usr/share/nginx/html`.
     - `index index.html;`: Specifies that `index.html` will be served as the default file when accessing the root URL.


4. **Push to Github repository**:
   - All the three files were to pushed to this repository.


## Deployment Phase

#### AWS EC2 Setup for running the Docker containers and pushing to AWS ECR

#### Steps to Initialize the AWS EC2 Instance and Deploying all the docker files which are Dockerfile, nginx.conf, index.html :

#### 1. Launch EC2 Instance

- **Step 1**: Sign in to the AWS Console.
  - Navigated to the [AWS Management Console](https://aws.amazon.com/console/), and logged in to my AWS account.
  
- **Step 2**: Navigate to EC2.
  - From the services menu, searcedh for **EC2** and selected **Launch Instance**.

- **Step 3**: Choose an Operating System.
  - Opted for **Ubuntu** as the OS.

- **Step 4**: Choose Architecture.
  - Selected **64-bit (arm)** architecture.

- **Step 5**: Choose Instance Type.
  - Chose **t4g.nano** for this case.

- **Step 6**: Configure Key Pair.
  - Set up key-pair login for the EC2 instance (download the `.pem` file for future access).

- **Step 7**: Configure Security Groups.
  - Allowed the following traffic:
    - SSH traffic
    - HTTPS traffic
    - HTTP traffic
  - Selected "Anywhere 0.0.0.0/0" for all.

- **Step 8**: Kept Storage as Default.

- **Step 9**: Review the Instance Summary.
  - Verified the summary on the right side of the screen before creating the instance.

- **Step 10**: Launch the Instance.
  - Launched the instance using **EC2 Instance Connect** instead of using the `.pem` file.

- **Note**: Downloaded the key pair (`.pem` file) for future access.

#### 2. Creating a Repository in AWS ECR

Find the attached SS for the same




#### 3. Installing docker on AWS EC2 Instance

Run the following commands on the EC2 instance:

```bash
sudo apt-get update
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
```

#### 4. Cloned the AWS-Containerization git repository on EC2 instance
```bash
git clone https://github.com/Aditya-rgb/AWS-Containerization.git

```
 
#### 5. Build Docker Container and Run

- **Step 1**: Build the Docker container.
  - Run the following command to build the Docker image with the tag `aditya-container:v1`:
    ```bash
    sudo docker build -t aditya-container:v1 .
    ```

- **Step 2**: List Docker images.
  - Check the available Docker images by executing:
    ```bash
    sudo docker images
    ```

- **Step 3**: Run the Docker container.
  - Execute the command below to run the container in detached mode, mapping port 3000 on the host to port 80 on the container:
    ```bash
    sudo docker run -d -p 3000:80 -t aditya-container:v1
    ```
  - Before testing the public IP on browser, go to security options of your EC2 instance and edit the inbound rule to hear from port 3000 too.

- **Step 4**: List running Docker containers.
  - To view the currently running Docker containers, use:
    ```bash
    sudo docker ps
    ```

#### 6. Install AWS CLI

- **Step 1**: Install AWS CLI.
  - Use the following command to install the AWS CLI:
    ```bash
    sudo snap install aws-cli --classic
    ```

- **Step 2**: Configure AWS CLI.
  - Set up the AWS CLI with your credentials by running:
    ```bash
    aws configure
    ```

#### 7. Log in to Amazon ECR

- **Step 1**: Get the login password for ECR.
  - Retrieve the login password using the command below:
    ```bash
    PASSWORD=$(aws ecr get-login-password --region us-west-2)
    ```

- **Step 2**: Log in to Docker with sudo.
  - If required, log in again using sudo:
    ```bash
    echo $PASSWORD | sudo docker login --username AWS --password-stdin <AWS-ACCOUNT-ID>.dkr.ecr.us-west-2.amazonaws.com
    ```


#### 8. Tagging and Pushing Docker Image to Amazon ECR

   - **Step 1**: The Docker image created earlier is tagged with the Amazon ECR repository URI to prepare it for pushing.
     - Tagging the Docker Image
     ```bash
     sudo docker tag aditya-container:v1 <AWS-ACCOUNT-ID>.dkr.ecr.us-west-2.amazonaws.com/aditya-docker-nginx:v1
     ```

   - **Step 2**: Listing Docker Images
     - The following command lists all the Docker images available on the local machine to verify the tagging.
     ```bash
     sudo docker images
     ```

   - **Step 3**: Pushing the Docker Image to Amazon ECR
     - The tagged Docker image is pushed to the specified Amazon ECR repository.
     ```bash
     sudo docker push <AWS-ACCOUNT-ID>.dkr.ecr.us-west-2.amazonaws.com/aditya-docker-nginx:v1
     ```

   - **Step 4**: Please find the Private repository AWS ECR link below along with the attached SS for it
     - AWS ECR private repo URL.
     ```bash
     975050024946.dkr.ecr.us-west-2.amazonaws.com/aditya-docker-nginx
     # OR
     IMG URL - 975050024946.dkr.ecr.us-west-2.amazonaws.com/aditya-docker-nginx:v1
     # OR
     https://us-west-2.console.aws.amazon.com/ecr/repositories/private/975050024946/aditya-docker-nginx?region=us-west-2

     # NOTE : Couldn't push to public repo due to some IAM policy error. Kindly Consider

## Testing Phase

### Accessing the Application

1. **Open the Application**:
   - Opened a web browser and navigated to IP address http://35.88.28.2:3000/
   - Verified that the HTML page loads correctly without errors.

2. **Checked Nginx Logs**:
   - Access the logs to ensure there are no errors during startup:
   ```bash
   docker logs <container_id>
   ```
3. **Test HTTP response**:
   - Used curl to test the HTTP response:
   ```bash
   curl -I http://35.88.28.2:3000/
   ```

## Summary
- The image `aditya-container:v1` was successfully tagged and pushed to the Amazon ECR repository `aditya-docker-nginx`.

  
## Contributing

We welcome contributions! To contribute:

1. Fork the repository.
2. Create a new branch for your feature or bug fix.
3. Commit your changes with clear messages.
4. Submit a pull request for review.

Make sure to follow the code style guidelines and include proper documentation for any new features.


## Contact

For any queries, feel free to contact me:

- **Email:** adityavakharia@gmail.com
- **GitHub:** [Aditya-rgb](https://github.com/Aditya-rgb/AWS-Containerization/tree/main)

You can also open an issue in the repository for questions or suggestions.


