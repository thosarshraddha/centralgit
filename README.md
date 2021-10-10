# Complete Automation Integrating Docker, Github with Jenkins Multibranch CI/CD Pipeline on AWS EC2

![](https://i.imgur.com/Y36pH4c.png)

## What is CI/CD?
- CI stands for Continous Integration
- CD stands for Continous Delivery or Continous Deployment

Let me explain through a example: 

When you make a commit and push sources to Git SCM(Source Code Management like Github, GitLab etc), then automatic builds and tests are run with the latest changes, this is knows as Continous Integration

With Continous Delivery, code changes are automatically built, tested, and prepared for a release to production. Continuous delivery expands upon continuous integration by deploying all code changes to a testing environment and put on a manual approval for adding them in production envirment.

Continous Deployment expands upon continuous delivery and pushed to production environment automatically without any approval.

![](https://d1.awsstatic.com/product-marketing/DevOps/continuous_integration.4f4cddb8556e2b1a0ca0872ace4d5fe2f68bbc58.png)

## What is Pipeline and MultiBranch Pipeline?

A DevOps pipeline is a set of automated processes and tools that allows both developers and operations professionals to work cohesively to build and deploy code to a production environment<br/><br/>
Jenkins MultiBranch pipeline allows us to automatically create a pipeline for each branch on your source control repository.<br/>Multibranch pipeline works using along with Jenkinsfile that is usually stored along with our source code inside a version control repository.<br/>

## Jenkins

Jenkins is an open source automation server.
The [Jenkins](https://www.jenkins.io) project was started in 2004 (originally called Hudson) by Kohsuke Kawaguchi, while he worked for Sun Microsystems. It was mainly developed for Continuous Integration, but in recent days, Jenkins orchestrates the entire software delivery pipeline — called continuous delivery and Continuous Deployment (CD). Presently, it is among one of the most popular Devops tools.
## Docker

[Docker](https://github.com/docker/docker) is a tool designed to make it easier to create, deploy, and run applications by using containers. Containers allow a developer to package up an application with all of the parts it needs, such as libraries and other dependencies, and deploy it as one package. By doing so, thanks to the container, the developer can rest assured that the application will run on any other Linux machine regardless of any customized settings that machine might have that could differ from the machine used for writing and testing the code.

It is a tool that is designed to benefit both developers and system administrators, making it a part of many DevOps (developers + operations) toolchains. For developers, it means that they can focus on writing code without worrying about the system that it will ultimately be running on. It also allows them to get a head start by using one of thousands of programs already designed to run in a Docker container as a part of their application. For operations staff, Docker gives flexibility and potentially reduces the number of systems needed because of its small footprint and lower overhead.
Here, we will be integrating Docker with Jenkins.

Pre-Requisites
- RHEL8 (you may install any other OS but steps will differ)
- Docker
- Jenkins
- MobaXterm SSH Client (or any other whichever you prefer)

Steps:
1. Deploy a AWS EC2 Instance of RHEL8-OS from AWS Console.<br/>
Make sure to add Inbound rules for 8080 port for Source 0.0.0.0/0

2. Execute these commands to Install Docker:
```
sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
sudo dnf install docker-ce
```

3. Execute these commands to Install Wget, OpenJDK, Jenkins:
```
sudo dnf update -y
sudo dnf install wget -y
sudo wget http://pkg.jenkins-ci.org/redhat-stable/jenkins.repo -O /etc/yum.repos.d/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key
sudo  dnf install -y java-11-openjdk-devel
sudo dnf install http://repo.okay.com.mx/centos/8/x86_64/release/daemonize-1.7.8-1.el8.x86_64.rpm
sudo dnf install -y jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

4. Give Jenkins user sudo priviledges:<br/>
```
vi -f /etc/sudoers
```
and add the following line in the bottom of file:<br/>
```
jenkins ALL= NOPASSWD: ALL
```

5. Lets Configure Jenkins now:<br/>
Open jenkins on web-browser using URL: ```http://aws_ip_address:8080```<br/>

    1. You will need to get the token from ```/var/lib/jenkins/secrets/initialadminpassword```.<br/>
    Do ```sudo cat /var/lib/jenkins/secrets/initialadminpassword``` and copy the text and paste that on Jenkins webpage.
    
    2. Select Install Suggested Plugins
    
    3. Create a first admin user
    
    4. Save and Finish
    
6. Create a Personal Access Token by visiting [here](https://github.com/settings/tokens). Set all permissions and set Expiration to No Expiration.<br/><br/>
And then go to Manage Jenkins->Configure System<br/>
In the Add GitHub Server:<br/>
Enter Name: ```Personal_Access_Token_USER```<br/>
API URL: ```https://api.github.com```<br/>
Add->Jenkins, select Kind->Secret Text<br/>
Enter Secret as your Personal Access Token which you created earlier.<br/>
ID (optional): ```b91c96c3-7a3f-4e08-b22d-c1100dd49eb9```<br/>
Enter same ID if you are going to use my Jenkinsfile from this project or edit the ID in the Jenkinsfile acccordingly.<br/><br/>
Go to Manage Jenkins->Manage Credentials->Jenkins->Global credentials(unrestricted)->Add Credentials<br/>
Username: Your GitHub Username<br/>
Password: Enter the Personal Access Token which was generated earlier<br/>
ID (optional): ```434b0b23-9deb-4ee6-85d4-43c4c23513bb```<br/>
Enter same ID if you are going to use my Jenkinsfile from this project or edit the ID in the Jenkinsfile acccordingly.<br/>

7. Download and Modify Jenkinsfile<br/>
**What it does?**<br/>
Well, Jenkinsfile is the ultimate pipeline file through which Jenkins will automatically detect the whole automation process and follow the steps mentioned in it.<br/><br/>
Download my [Jenkinsfile](https://github.com/Amsal1/devops_ci_cd/blob/master/Jenkinsfile)<br/>

Replace my git repo url with yours throughout the file. Also change UserIdentity block with your GitHub Details in class GitSCM of Code Checkout<br/>
Push this modified Jenkinsfile to your both branches in your repo. It would be better if you add it in master branch and create dev branch using that but you may only do this on a new repo. For a older repo, you may need to use combinations of few git commands including git reset, git rebase, git checkout and git branch. Text me if you face any diffcuilties😄<br/>
Also remember, I am using master and dev branch. If you're using any any other branch instead of 'dev' then, make changes accordingly in Jenkinsfile.<br/>
Also open inbound ports 82 and 83 in AWS EC2 Console like you did for port 8080 in Step 1<br/>


8. Select New Item:<br/>
Enter Item Name: ```git_job_pipeline```<br/>
Select Multibranch pipeline and click OK<br/><br/>

Select Add Sources->GitHub<br/>
Select and set Credentials and set your repo name. <br/>
In my case: ```https://github.com/Amsal1/devops_ci_cd/```<br/>
Click Save and it will done rest😄<br/>
