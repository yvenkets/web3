# web3

++++++++++++++++++++++++++++++++++++++++++
Code Commit, Code deploy, Code pipeline.
++++++++++++++++++++++++++++++++++++++++++

Create two IAM roles.

EC2 instance :
IAM roles >> 
name: codecommitec2role
role: AmazonEC2RoleforAWSCodeDeploy

CodeDeploy:
IAM roles >> 
name: codecommitcodedeployrole
role: AWSCodeDeployRole




1. Install git in localmachine
2. Create a repository in codecommit.
3. Clone the  code commit repo to localmachine
4. Update the neccessary file to repo
5. Push the code to codecommit using git.
6. Launch the ec2 instance with suffient IAM roles
7. install codedeploy agent in ec2 instance
8. Create Code deploy in AWS
7. Create Codepipeline


install git in localmachine
++++++++++++++++++++++++++
yum install git -y


Creating repository:
+++++++++++++++++++
AWS >> services >> Code commit >> 
Repository name* = testrepo1
Description = testrepo1
>> create Repository >>skip next step >> Choose ssh method.


#mkdir /project/codecommit
#cd /project/codecommit/

Follow below steps to clone the repository of codecommit repo to localmachine
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++
https://docs.aws.amazon.com/codecommit/latest/userguide/setting-up-ssh-unixes.html?icmpid=docs_acc_console_connect


#git clone ssh://git-codecommit.us-east-2.amazonaws.com/v1/repos/testrepo1
This is the link which shown while creating repo to clone.

Move to the cloned repo folder
#cd testrepo1

Now iam inside /project/codecommit/testrepo1

#git init

Create the below files appsec.yml, install_dependencies, stop_server, start_server
++++++++++++++++++++++++++++++++++++++++++++++


#vi appspec.yml 

version: 0.0
os: linux
files:
  - source: /index.html
    destination: /var/www/html/
hooks:
  BeforeInstall:
    - location: scripts/install_dependencies
      timeout: 300
      runas: root
    - location: scripts/start_server
      timeout: 300
      runas: root
  ApplicationStop:
    - location: scripts/stop_server
      timeout: 300
      runas: root

      
#vi index.html
This is webpage for codecommit test
      
#mkdir scripts

vi scripts/install_dependencies 
#!/bin/bash
yum install -y httpd

vi scripts/start_server 
#!/bin/bash
service httpd start


vi scripts/stop_server 
#!/bin/bash
isExistApp = `pgrep httpd`
if [[ -n $isExistApp ]]; then
    service httpd stop
fi

To add the files to repo
#git add.

To check the status 
#git status

To commit the codes to localmachine
#git commit -m 'First commit'

To push the codes to AWS code commit
#git push origin master



Launch a Ec2 instance AWSlinux  with IAM role codecommitec2role which we created first.
security grup : ssh, httpd


Install code deploy agent inside the ec2 isntance.

sudo yum -y update
sudo yum -y install aws-cli
sudo yum install ruby -y
sudo yum install wget -y 
cd /home/ec2-user
wget https://aws-codedeploy-us-east-1.s3.amazonaws.com/latest/install
chmod +x ./install
sudo ./install auto
sudo service codedeploy-agent start



Create CodeDeploy:
++++++++++++++++++++++
AWS >>Custom deployment>>  Skip walkthrouh 
Application name = Codedeploytest
Compute Platform = Ec2/Onpremises
Deployment group name =  Codedeploytestgroup

Environment configuration >> AWS ec2 instance >> 
select the tag of the server we created.


Deployment configuration >> CodeDeployed.AllatOnce


Advanced >> Service role >> codecommitcodedeployrole 
This is the role which we created first.


Create Codepipeline
++++++++++++++++++++

AWS >> Codepipeline >> 
Pipeline name = codecommitpipeline
>> next
Source Provider >> AWS code commit 

Repository name = testrepo1
Branch name = master

>> next

Build >> Build provider = No Build

Deploy >> Deployment provider = AWSCodeDeployRoles

AWS CodeDeploy >> 
Application name* = Codedeploytest
Deployment group* = Codedeploytestgroup
>>next

AWS Service Role >> Role name* >> Create Role (click on Create role)
New page open >> Select the default no change >> Click on allow.
>>next

>> Create pipeline
    
