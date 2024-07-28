
```markdown
# Containerized Web Application Deployment and CI/CD Pipeline Implementation

## Team Members
- **Jashwanth Raj Gowlikar** (G01356160)

## Steps to Create a Docker Image and Push it to Docker Hub

1. **Install Docker**: Install Docker on your system using the following command:
   ```bash
   sudo apt-get install docker.io
   ```

2. **Convert Your HTML File to .war File**: Convert your `index.html` file to a `.war` file using the command:
   ```bash
   jar -cvf student.war -C webapp/
   ```
   - `student.war` is the name of the file.
   - `webapp` is the path of the folder that contains the `index.html` file.

3. **Create a Dockerfile**: In your VS Code, create a new file named `Dockerfile` and write the following commands to publish your `.war` using a Tomcat server image.

4. **Build the Docker Image**: Create a Docker container using the following command. Ensure you have the `student.war` and `Dockerfile` in the same folder.
   ```bash
   docker build -t jashwanthraj/webapp:1.0 .
   ```
   - `jashwanthraj/webapp` is the name of the repository on Docker Hub.
   - `1.0` is the version of the image.

5. **Run the Docker Build**: Test the Docker build by using the following command:
   ```bash
   docker run -it -p 8888:8080 jashwanthraj/webapp:1.0
   ```

6. **Open in Browser**: Open your browser and go to `localhost:8888/student` (name of your war file). Ensure you have Tomcat installed and it's running on your system.

7. **Login to Docker Hub**: To push the Docker image to Docker Hub, first log in using the command:
   ```bash
   docker login -u <docker_username>
   ```

8. **Push the Image**: Push the image using the command:
   ```bash
   docker push jashwanthraj/webapp:1.0
   ```

9. **Verify on Docker Hub**: Go to Docker Hub and verify that the image has been pushed to the repository.

## Steps to Create Rancher Server on AWS

1. **Launch EC2 Instances**:
   - Go to your AWS dashboard and search for EC2.
   - Go to the Instances section and create two EC2 instances: one as `rancher-server` and the other as `worker` with the same configurations.
   - Use medium size machines (t2 or t4), create a key pair, and configure the security group to allow HTTP (port 80), HTTPS (port 443), and SSH (port 22).
   - Change the name of one instance to `worker` to avoid confusion.

2. **Edit Security Group**:
   - Edit the inbound rules to allow "custom TCP at port 8080" and "allow all TCP".
   - Create two Elastic IP addresses and associate them with the Rancher and Worker instances.

3. **Connect to Instances**:
   - Use EC2 instance connect to open a web-based CLI connected to the instances.
   - Install Docker on both instances using:
     ```bash
     sudo apt install docker.io
     ```
   - Check if Docker is installed using:
     ```bash
     docker -v
     ```
   - Give super user permissions to Docker using:
     ```bash
     sudo usermod -aG docker ubuntu
     ```

4. **Start Rancher**:
   - On the Rancher EC2 instance, run the command to start Rancher:
     ```bash
     sudo docker run --privileged -d --restart=unless-stopped -p 80:80 -p 443:443 rancher/rancher
     ```
   - Go to the public IP address of the Rancher EC2 instance to launch Rancher.
   - Follow the on-screen instructions to get the password and paste it on the Rancher page.

5. **Create a Cluster**:
   - Create a password for the admin user if not using the suggested password.
   - Click on create to create a cluster.
   - Select "RKE1" and click on custom to create a Kubernetes cluster.
   - Name the cluster and click next. Select all the node options and copy the command to run on the worker EC2 instance.

6. **Register Node**:
   - Wait until the command executes and you'll see a green label "node has been registered" on the Rancher page and click done.
   - Wait until the cluster is provisioned and the status changes to active.

7. **Deploy Docker Image**:
   - Click on the cluster you created and go to workloads, then deployments to create a deployment.
   - Name your deployment and provide the container image from your Docker Hub `<username>/<application_name>:version`.
   - Add a node port, name it, and enter the private container port as `8080`.
   - Set the number of pods to 3 and click create.
   - Create another deployment as a load balancer with the same image details and select never to pull the image.
   - Under add port, select load balancer and enter the private container port as 8080 and listening port as the endpoint port.

8. **Access the Application**:
   - Use the link under the target named load 8080/tcp and append the name of the `.war` file.

## Creating a CI/CD Pipeline using Git and Jenkins

1. **Create a Git Repository**:
   - Upload your files (Dockerfile, index.html, .war file) into the Git repository using CLI or the upload files link.

2. **Launch a New EC2 Instance**:
   - Create a new instance and install Docker using the steps above. Elastic IP is not required.

3. **Install Jenkins**:
   - Install Jenkins using the following commands:
     ```bash
     sudo apt install fontconfig openjdk-17-jre
     sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
     echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
     sudo apt-get update
     sudo apt-get install jenkins
     ```
   - Check the status of Jenkins using:
     ```bash
     sudo systemctl status jenkins
     ```
   - Go to `localhost:8080` in your browser to access the Jenkins page.

4. **Setup Jenkins**:
   - Get the password from the CLI of the server using:
     ```bash
     sudo cat /var/lib/jenkins/secrets/initialAdminPassword
     ```
   - Create a user and password, then select to install suggested plugins.

5. **Install kubectl**:
   - Install kubectl using:
     ```bash
     sudo snap install kubectl --classic
     ```
   - Copy the kubeconfig from Rancher by going to the cluster created.
   - Create a config file at location `~/.kube/` using:
     ```bash
     vim config
     ```
   - Copy the kubeconfig to the config file and verify that kubectl is working by running:
     ```bash
     kubectl config current-context
     ```

6. **Configure Jenkins**:
   - Go to the Jenkins dashboard, click on your username, then credentials.
   - Click on system, then global credentials and add your GitHub user details and Docker user details.
   - Install Docker, Docker Pipeline, Docker API, and CloudBees Docker plugins on Jenkins and restart Jenkins.

7. **Create a Jenkinsfile**:
   - Create a Jenkinsfile in VS Code with the pipeline details.
   - Add kubeconfig as an environment variable with the path to the config file.
   - Upload the Jenkinsfile to GitHub.

8. **Create a Jenkins Job**:
   - Create a new job in Jenkins, name it, and select pipeline from the options.
   - Under poll SCM, enter `* * * * *` to check for changes in Git every minute.
   - Under pipeline, select pipeline from SCM and Git under SCM, and provide the repository details.
   - Select the Git credentials from the list and apply.

9. **Trigger Builds**:
   - Any changes to the Git files will trigger the build.
   - Check the changes to the HTML page by going to the same link of the load balancer.
