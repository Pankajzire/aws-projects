
# Deploy simple website on AWS using NGINX, AWS Codecommit, Codebuild, Codedeploy, Codepipeline, Artifact, EC2, S3, IAM.

## Step 1 : Create Repositories on AWS using CodeCommit

 1. Open Developer Tools in AWS console 
 2. Go to CodeCommit > Repositories > create 
 3. Give name - Demo-Repository
 4. Create

## Step 2 : Create new IAM user

 1. Create new IAM user > Click on user 
 2. Add following policy permission : AWSCodeCommitPowerUser
 2. Security credential > HTTPS Git credential for AWS CodeCommit  > Generate Credentials> download Credentials 
## Step 3 : Clone Repositories to local

 1. Go to CodeCommit > Repositories > select Repositories > clone URL > HTTPS
 2. create new folder on your local and open it with VS-Code
 3. paste URL in terminal - git clone "URL"
 4. new prompt will open enter the Credentials that you downloaded earlier 
 5. Repositorie is cloned with local

 ## Step 4 : Add files to folders

 1. index.html 

        <!DOCTYPE html>
        <h1>Welcome to my website</h1>

2. buildspec.yml

        version: 0.2
        phases:
          install:
            commands:
              - echo Installing NGINX
              - sudo apt-get update
              - sudo apt-get install nginx -y
          build:
            commands:
              - echo Build started on 'date'
              - cp index.html /var/www/html/
          post_build:
            commands:
              - echo Configuring NGINX

        artifacts:
          files:
            - '**/*'



3. appspec.yml

        version: 0.0
        os: linux
        files:
          - source: /
            destination: /var/www/html
        hooks:
          AfterInstall:
            - location: scripts/install_nginx.sh
              timeout: 300
              runas: root
          ApplicationStart:
            - location: scripts/start_nginx.sh 
              timeout: 300
              runas: root 

4. create new folder scripts and add following files to it

* start_nginx.sh

      #!/bin/bash

      sudo service nginx start

* install_nginx.sh

      #!/bin/bash

      sudo apt-get update
      sudo apt-get install -y nginx


5. Run following Git commands:

       git add .
       git commit -m "added new index file, buildspec, appspec, scripts"
       git push



## Step 5 : Build project using Codebuild

1. Go to Codebuild > build project > create build project
2. project name - DevOps-demo
3. source provider - CodeCommit 
4. Repository - Demo-Repository
5. branch - master
6. Environment image - Managed image 
7. operating system - Ubuntu
8. Runtime - Standard
9. image - select latest image
10. service role - new service role
11. buildspec - use a buildspec file
12. artifact - 
* type - S3
* bucket name
* name - build_output
* Artifacts packaging - .zip
13. create
14. start build

## Step 6 : Build Application using Codedeploy
#### Create a service role "code-deploy-service-role" for Codedeploy add following policies permissions
* AmazonEC2FullAccess
* AmazonS3FullAccess
* AWSCodeDeployRole
* AmazonEC2RoleforAWSCodeDeploy
* AWSCodeDeployFullAccess
* AmazonEC2RoleforAWSCodeDeployLimited
#### Create a role "ec2-code-deploy-role"  for EC2 and add following policy permissions
* AmazonEC2FullAccess
* AmazonS3FullAccess
* AWSCodeDeployFullAccess

####  launch EC2 instance

1. launch an EC2 instance Name - demo-instance
2. attach "ec2-code-deploy-role" to it 
3. connect to it
#### Run following commands to Install AWS CodeDeploy Agent on Ubuntu EC2

*      vim insatll.sh
* paste following commands in it 

      #!/bin/bash 
      # This installs the CodeDeploy agent and its prerequisites
       on Ubuntu 22.04.  
       sudo apt-get update 
       sudo apt-get install ruby-full ruby-webrick wget -y 
       cd /tmp 
       wget https://aws-codedeploy-ap-south-1.s3.ap-south-1.amazonaws.com/releases/codedeploy-agent_1.3.2-1902_all.deb 
       mkdir codedeploy-agent_1.3.2-1902_ubuntu22 
       dpkg-deb -R codedeploy-agent_1.3.2-1902_all.deb codedeploy-agent_1.3.2-1902_ubuntu22 
       sed 's/Depends:.*/Depends:ruby3.0/' -i ./codedeploy-agent_1.3.2-1902_ubuntu22/DEBIAN/control 
       dpkg-deb -b codedeploy-agent_1.3.2-1902_ubuntu22/ 
       sudo dpkg -i codedeploy-agent_1.3.2-1902_ubuntu22.deb 
       systemctl list-units --type=service | grep codedeploy 
       sudo service codedeploy-agent status

*     Esc :wq

*     bash install.sh

    

#### Start building Application

1. Go to Codedeploy > Applications 

* Create application 
* Application name - demo-app
* Compute platform - EC2/On-premises
* create

2. Create deployment group 

* deployment group name- demo-group
* Service role - code-deploy-service-role
* Deployment type- In place
* Environment configuration - Amazon EC2 instances
* Name - demo-instance
* Install AWS CodeDeploy Agent - never
* Enable load balancing - uncheck
* create deployment group

3. Create deployment

* deployment group - select deployment group 
* Revision type - My Application is stored in Amazon S3
* Revision location - paste the S3 URI of artifact
* Revision file type - .zip
* create deployment

## Create Codepipeline for CICD

1. Go to  Codepipeline > Create pipeline
2. Pipeline name - demo-pipeline
3. Service role - New service role
4. Source provider - CodeCommit
5. Repository name - Demo-Repository
6. Branch name - master
7. Change detection options - AWS CodePipeline
8. Output artifact format - CodePipeline default
9. Build provider - AWS Codebuild
10. Project name - DevOps-demo
11. Environment variables - Single build
12. Deploy provider - AWS codedeploy
13. Region - mumbai
14. Application name - demo-app
15. deployment group - demo-group
16. create pipeline