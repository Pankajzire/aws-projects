
# Creating S3 bucket using CloudFormation from Jenkins PipeLine and Github 

##  Jenkins installation 

Install Jenkins on AWS EC2

*  Jenkins is a self-contained Java-based program, ready to run out-of-the-box, with packages for Windows, Mac OS X and other Unix-like operating systems. 
 *  As an extensible automation server, Jenkins can be used as a simple CI server or turned into the continuous delivery hub for any project.

#### Prerequisites
* EC2 Instance With Internet Access
* Security Group with Port 8080 open for internet
* Java 11 should be installed
* Install Jenkins
* You can install jenkins using the rpm or by setting up the repo. We will set up the repo so that we can update it easily in the future.

#### After doing ssh into instance use following cammands:

* Get the latest version of jenkins from:

1.     https://pkg.jenkins.io/redhat-stable/ 
 

2.     sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo

3.     sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key

4. To install epels

       amazon-linux-extras install epel 

5. To install  Java

       amazon-linux-extras install java-openjdk11  

6. Repository that provides 'daemonize'
 
       yum install epel-release  

       yum install java-11-openjdk-devel

7.     yum install jenkins

8. Start jenkins service

       service jenkins start

###  Setup Jenkins to start at boot

*  chkconfig jenkins on Accessing Jenkins By default jenkins runs at port 8080 
* Add a new rule in your security group on port 8080
*  You can access jenkins at:

       http://YOUR-SERVER-PUBLIC-IP:8080

* Configure Jenkins
* The default Username is admin
* Grab the default password Password Location:

      cat /var/lib/jenkins/secrets/initialAdminPassword

* yum install git
*  add git, github, git server, GitHub Integration, GitLab Authentication, Gitlab API, GitHub Authentication, Pipeline: GitHub in plugins
* Change admin password
* Admin > Configure > Password
* Configure java path
* Manage Jenkins > Global Tool Configuration > JDK
* Create another admin user id
* Test Jenkins Jobs 

## Attach IAM role to your EC2 Instance

1. Open IAM Console
2.  Click on Roles > create roles > AWS service > EC2 
3.  Add these two policies: 
* AWSCloudFormationFullAccess 
* AmazonS3FullAccess
4. Open EC2 dashboard select instance and attach role

## Add these both files to your Github Repository

1. Jenkins file (jenkinsfile):

       pipeline {
       agent any
       stages {
        stage('Submit Stack') {
            steps {
            sh "aws cloudformation create-stack --stack-name s3bucket --template-body file://simplests3cft.json --region 'ap-south-1'"
              }
             }
            }
            }


2.  S3 file (simplests3cft.json) :

        {
        "AWSTemplateFormatVersion":"2010-09-09",
        "Resources": {
        "S3Bucket": {
            "Type": "AWS::S3::Bucket"
        }
        },
        "Outputs": {
        "BucketName": {
            "Value": {
                "Ref": "S3Bucket"
            },
            "Description": "Name of the sample Amazon S3 bucket."
        }
        }
        }

   




## Jenkins Github integration 

1. Open Github on your browser
2. Click on Profile > Settings >  Developer settings >  Personal access tokens > Generate new tokens
* Note- jenkins-cicd
3. select :
* repo
* workflow
4. Generate token
5. copy that token and save it later you cannot access it


## Run CloudFormation Using Jenkins PipeLine

1. select new item 
2. name- clf-s3-pipeline > pipeline
3. pipeline > Definition > pipeline script from SCM 
4. SCM- Git
5. Repository URL- paste the Github url of Repository where you store the simplests3cft and jenkinsfile
6. Credentials > Add > Jenkins 
7. Kind - Username and password
8. add Username and Password- personal access token that you created
9. branch- main
10. create pipeline
11. build
12. check in CloudFormation stack is created


## Clean up

* delete stack
* delete EC2 instance