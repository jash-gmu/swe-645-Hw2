Containerized Web Application Deployment and CI/CD Pipeline Implementation
Team mates :
Jashwanth Raj Gowlikar(G01356160)
1.Steps to create a docker image and push it to docker hub:
• Install docker on your system using the following commad. “Sudo apt-get install docker.io”
• Convert your index.html file to .war file using the command “jar -cvf student.war -C webapp/”, where
student .war is the name of the file and web app is the path of the folder that contains the index.html
file.
• In your vs code create a new file named “Dockerfile”. And write the following commands to publish
our .war using tomcat server image.
• Create a docker container using the following commad and make sure you have the student.war and
dockerfile in a same folder.
• “docker build -tag jashwanthraj/webapp:1.0” where “jashwanthraj/webapp” is the name of the
repository on the docker hub. And 1.0 is the version of the image.
• Run the Docker build to make sure it works by using the following command “docker run -it -p
8888:8080 jashwanthraj/webapp:1.0”
• Open your browser and got to localhost:8888/student(name of your war file ) , make sure you have
tomcat installed and its running on your system.
• To push the docker file to the docker hub you have to login first using the command “docker login -u
<docker _ username> ”
• To push the image use the commad “docker push jashwanthraj/webapp:1.0 ”
• Go to docker hub and verify that the image has been pushed to the repo.
2.Steps to create rancher server on AWS.:
• Go to your aws dashboard and search for ec2
• Got to instances section and create two ec2 instances one as rancher-server and other worker with
same configurations .
• The configurations include any medium size machine (t2 or t4) and choose to create a key pair and
make sure you create a security group and enable http(port 80), https(443),ssh(22) in the
configuration page of the ec2 and click on launch instance.
• Make sure you change one of the instances name as “worker” to avoid confusion.
• Go to security group section and edit the inbound rules and allow the “custom tcp at port 8080” and
“allow all tcp” . make sure the security group you are editing is associated with the instances you
create.
• The reason of editing the inbound rules is we are going to access the resources from these ports.
• We also have to create two elastic ip addresses and associate them with the rancher and worker
instances ,as the ip address of the instances change every time they reboot, which causes the
communication problems with rancher and the worker instances.
• Connect to the instances using the “EC2 instance connect” which opens an web based cli
connected to the instances.
• Install docker using the command on both instances “sudo apt docker.io” and also check if its
installed using “docker -v” which should return the docker version installed.
• Also give the super user permissions to docker using “sudo usermod -aG docker ubuntu”
• In the rancher ec2 instance run the command to start rancher “$ sudo docker run --privileged -d --
restart=unless-stopped -p 80:80 -p 443:443 rancher/rancher”
• Go to the public ip address of the rancher ec2 instance and it should launch rancher.
• Copy the command on screen and run it on the ec2 connect of the rancher ec2 to get the password
and paste it in the rancher page
• Create a password for the admin user if you don’t want to use the suggested password.
• You should land on the rancher homepage and click on the create to create a cluster.
• Make sure you select “RKE1” and click on custom to create a Kubernetes cluster.
• Give the name to the cluster and click on next . and select all the node options and copy the command and
run on the worker Ec2’s instance connect
• Wait until the command has executed and you’ll see a green label with “node has been registered” on
rancher page and click done .
• Wait until the cluster is provisioned and the status changes to active.
• To deploy the docker image click on the cluster you have created and click on workloads and click on
deployments to create a deployment.
• Give an unique name to your deployment and give the name of the container image from your docker hub
<username>/<application name>:version
• Add a node port and give it a custom name and enter private container port as “8080” .
• Make sure you enter 3 in the number of pods section as we want 3 pods to be deployed and click on create.
• Create a another deployment as load balancer with same image details and select never to pull image.
• Under add port select loadbalancer and enter the private container as 8080 and listening port as the port
number that is at end point .(in my case its 3175)
•
• You can access the static page using the link under the target named load 8080/tcp , enter the name of the
war file at the end of the link /name of the war file
•
3.Creating a CI/CD Pipeline using git and Jenkins :
• Create a Git repo and upload your files(docker, index.html, .war file) into the git using the cli or the upload
files link.
• Create a new instance using the steps above and install docker in the instance. You don’t need to assign
elastic address to the instance.
• install Jenkins in the instance using the commands:
sudo apt install fontconfig openjdk-17-jre
java -version
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
/etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install Jenkins
• check the status of the Jenkins using “ sudo systemctl status jenkins”
• go to localhost:8080 in your browser and you should land on Jenkins page.
• Get the password from the cli of the server using the command “sudo cat
/var/lib/jenkins/secrets/initialAdminPassword”.
• Create user and password and select the install suggested plugins.
• Install kubectl using command “sudo snap kubectl --classic”
• Copy the kubeconfig from rancher, going to the cluster created .
• Create a config file at location ~/.kube/ using ‘vim config’ at the location
• Copy the kubeconfig to the config file just created .
• Verify that the kubectl is working by running ‘kubectl config current-context’ and getting the name of your
cluster
• Go to the Jenkins dash board and click on your username on top and then click on credentials
• Click on system and then click on global credentials and add your github user details and docker user details
and give them appropriate id.
• Click on configure and manage plugins and install docker , docker pipeline, docker api and cloud bees
docker on Jenkins and restart the Jenkins .
• Create a Jenkins file in vs code with the following details.
•
• Make sure you add kubeconfig as an environment variable which has the path to the config file.
• Upload the Jenkins file to the git hub.
• In Jenkins create a new job and give it a custom name. and select the pipeline from the given options.
• Under poll scm enter “* * * * * ” which means we want check for changes in the git everyminute.
• Under pipeline select pipeline from scm and git under scm give the details of the repository
•
• Select the credentials of git from the list and click on apply.
• Making any change to the git files will trigger the build
•
• You check the changes to the html page by going to the same link of the load balancer.
