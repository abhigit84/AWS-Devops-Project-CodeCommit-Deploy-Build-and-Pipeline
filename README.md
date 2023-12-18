Code Commit and Code Build:
Codecommit-like github.
Code Build-CI part and it will pick from Codecommit like jenkins and will store  artifact like S3 storage.
Code Deploy-CD part and takes from artifactory or S3 and will deploy on ec2/lambda/ecs.
Code pipeline-Pipeline having CICD and having stages like jenkins.

1)CodeCommit:-

Go to CodeCommit-Repository-Name-Create Repository-Optional to select codeguru which is like sonarqube.

Now root user cannot configure ssh/https so create user.

IAM-User-Create User-name-Provide user access to AWS Mgmt console-I want to Create and IAM User-autogenerate password-show password-User must create a new password at next sign-in-
Create User(this user has only 1 policy attached iamuserchangepassword)-Download and save credentials

Click on user created-Security Credentials tab-HTTPS Git Credentials for Code Commit-Generate Credentials-Download and save credentials

Now on Code Commit-Select Repository-Clone Url-Clone HTTPS(from clone url dropdown select this option)

copy url-
https://git-codecommit.us-east-1.amazonaws.com/v1/repos/demo-app

This url will be used to clone the repo in local.

IAM-User-select your user-Add permissions-attach policies directly-AWSCodeCommitPowerUser-add policies

Now in VS code in local enter:

git clone https://git-codecommit.us-east-1.amazonaws.com/v1/repos/demo-app
Put CodeCommit username and password in dialogbox that comes.

Copy and create any sample code and file named index.html inside the empty repository copied.

ON VS Code:

cd demo-app
git status
(it will show 1 untracked file as index.html)
git add .
git commit -m "added indexhtml file"
git push origin master

The above command pushes code into Code Commit repo.
Now when u refresh or see in codecommit repo u will see index.html

Create new branch:

git checkout -b dev
modify index.html
As this dev branch has all files of master it has index.html and so add and commit changes and push to dev branch and so new branch dev gets created on CodeCommit.
git add .
git commit -m "added changes"
git push origin dev

Now in CodeCommit dev branch gets created.
Now u can merge the changes done on dev to master branch through pull request just like github
select dev branch on codecommit and do PullRequest
Give any description-Create pull request-Merge-It will show 3 different options-select first(fastforward merge) and then delete dev after merging checkbox will delete dev after merging-Merge Pull request

Now u will see dev not there and on master index.html is updated.

You can see in left hand pane-approval rule templates wherein u can set how many people required to approve pull requests after creating template (just like github).

2)CodeBuild:-

Codebuild-Build projects-Create Build Project-Project name-source-source 1-Primary-Source Provider-AWS Code Commit-Repository-ur repository select-branch-master-
Enviroment-OS-ubuntu-
Runtime-Standard-Image-aws/codebuild/standard:7.0
ServiceRole-You create this as Codebuild need policies attached to access other services so this is already created with name -codebuild-demo-app-build-service-role
Buildspec-Use a buildspec file
On VS code:-
Now create a buildspec.yaml file which basically contains steps of building and do in dev branch
git add .
git commit -m "added buildspecyam file"
git push origin dev

Now Come back to Codebuild where u left-

Buildspec-Use a buildspec file-if u used name as buildspec.yaml then no need to select this as it will automatically pick uo this file
Create BuildProject-Start Build-You are running master branch-You can see now in phase details-it will fail at download source step as buildspec.yml does not exist in master branch.

Create pull request-
destination-master-source-dev-Title- after seeing changes-Create pull request-Merge-Merge Pull Request
Now u can see buildspec.yaml added in master branch
Now go to code build-after selecting ur build-Retry Build 
Build will succeed.

Change or set location of artifacts to s3.

Go to build-edit-artifacts-Type-Amazon s3-bucketname-select bucket name

......dont try this................
Name-artifacts.zip(this is compressed zip folder where u want ur code file to get stored as specified in buildspec.yml)
Path-code-output-demo-app(this is folder name that u created inside s3 bucket,if not there go to s3 and create a folder inside it)
Update Artifacts

.......dont try above......................


dont create folder and all inside s3 for storing .zip file as deployemnt failing in next day video.
so Name-artifacts.zip and dont put any path.
Update Artifacts

Now when u start build-all build files will go this s3 location.



Code Deploy and Code Pipeline:



Go to CodeDeploy-Applications-Create application-Choose Platform-EC2(it will show ec2/lambda/ecs)
Create deployment group(on which deployment will be done)-deployment name-service role-attach a service role(create a service-use case will be Code deploy and not ec2 with these acceses or create a new one with these services.) and mention its arn 
AmazonEc2fullaccess
AmazonEC2RoleforAWSCodeDeploy
AmazonS3FullAccess
AWSCodeDeployFullAccess
AWSCodeDeployRole
AmazonEC2RoleforAWSCodeDeployLimited
AWSCodeDeployRoleForECS


Create an EC2 Instance

Deployment type-in place

Enviroment configuration-Amazon EC2 Instance-Key-Name-Values-Put name of ec2 instance created
Install AWS Deploy Code agent-Never
Now this agent is used to deploy actual code on your ec2.So setting up this agent using shubham blog code as below.Just run this on ec2 instance and change region accordingly.
On EC2:-
vim install.sh
......................................................................................
#!/bin/bash 
# This installs the CodeDeploy agent and its prerequisites on Ubuntu 22.04.  
sudo apt-get update 
sudo apt-get install ruby-full ruby-webrick wget -y 
cd /tmp 
wget https://aws-codedeploy-us-east-1.s3.us-east-1.amazonaws.com/releases/codedeploy-agent_1.3.2-1902_all.deb 
mkdir codedeploy-agent_1.3.2-1902_ubuntu22 
dpkg-deb -R codedeploy-agent_1.3.2-1902_all.deb codedeploy-agent_1.3.2-1902_ubuntu22 
sed 's/Depends:.*/Depends:ruby3.0/' -i ./codedeploy-agent_1.3.2-1902_ubuntu22/DEBIAN/control 
dpkg-deb -b codedeploy-agent_1.3.2-1902_ubuntu22/ 
sudo dpkg -i codedeploy-agent_1.3.2-1902_ubuntu22.deb 
systemctl list-units --type=service | grep codedeploy 
sudo service codedeploy-agent status
...........................................................................................

Now we need appsec.yml to deploy app just like builspec.yml was used for building code.
So see 2 scripts and appspec.yml
Now we go to Vs code and do
git add .
git commit -m "added appspec and 2 scripts"

Now these all added in codecommit.
then we run Build again by Start build after going in Code Build.

Now go to IAM-Create a role for EC2 service so that ec2 can access s3,coddeploy etc and update role in ec2
attach these policies:-
AWSCodeDeployFullAccess
AmazonEC2FullAccess
AmazonS3FullAccess

Now go to ec2 instance console-sudo service codedeploy-agent restart
so that it takes all permissions given
sudo service codedeploy-agent status
it should be in running state.


Now in Codedeploy-application-select your application-select your deployment group-Create Deployment-copy zip file uri from s3 that u created in earlier day-s3://nomad12/artifacts.zip-Revision file type-.zip
dont create folder and all inside s3 for storing .zip file as deployemnt failing.
Create Deployment

Now you can see at public ip of instance on browser that nginx is deployed and index.html contents are displayed.

Note:To run anything in nginx proxy tool we have to place html file in /var/www/html folder.
Also run as root means no permission needed for script run in appspec.yml
in buildspec.yml artifacts:
  files:
    - '**/*' means all files and all folders should be copied to artifacts.zip folder in s3.

CodePipeline:

like a complete automated CICD pipleine

Pipeline-Pipelines-Create Pipeline-New service role(which is used to connect Pipeline to talk to various services)-Next-
Source Provider-AWS Code commit-Repository name-select ur repo-branch-master-Change detect options-AWS CodePipeline-Code Pipeline default-
Next-Build-Build provider-AWS Code Build-project name-select ur build name-single build-next
Deploy-DeployProvider-AWSCodeDeploy-application name-put urs-deployment group-select urs-
Create Pipeline.

Now make some changes in repo add and commit and
then do Release Changes in Code Pipeline then it will trigger.
You have to click on some info box that comes do polling for detecting any changes in repo so do it and then everytime build will automatially trigger everytime there is push to repo from local after some changes.








