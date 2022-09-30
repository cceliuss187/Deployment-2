<h1>Testing stage of the CI/CD pipeline deployment 2 </h1>

<h2>(1) Set up and configure your Jenkins environment.</h2>


- INSTALL JENKINS ON AN EC2 IF YOU HAVEN'T ALREADY


```
sudo apt update
sudo apt ugrade -y
sudo apt -y install openjdk-11-jre
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee \
/usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
/etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get -y install jenkins
sudo systemctl start jenkins
systemctl status jenkins >> ~/file.txt
```
------------------------------------------------------------------------------------------------------------------------------
- UNLOCK JENKINS AND PROCEED WITH THE SETUP.

To unlock jenkins, cat the file below and copy & paste the password.
(This is done to ensure that Jenkins is securely set up by the administrator.)
```
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
------------------------------------------------------------------------------------------------------------------------------

- IF YOU ALREADY HAVE JENKINS ON AN EC2.
Activate the Jenkins user on that EC2. 
```
sudo passwd jenkins
sudo su - jenkins -s /bin/bash
```
------------------------------------------------------------------------------------------------------------------------------

- CREATE A JENKINS USER IN YOUR AWS ACCOUNT

	- Navigate to IAM in the AWS console. Next click on the Users option in the Access management. 
	- Select add user. Next, the username can be EB-user. 
	- Then select Programmatic access and then next.
	- Select "Attach existing policies directly" and select administrator access. Now select Next for this page and the next page. 
	- Finally, create the user and then copy and save the "access key ID" and the "secret access key".

------------------------------------------------------------------------------------------------------------------------------

- INSTALL AWS CLI ON THE JENKINS EC2 AND CONFIGURE:
 
```
sudo apt install unzip
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" 
ls
unzip awscliv2.zip 
sudo ./aws/install
sudo su - jenkins -s /bin/bash
aws configure
```
	- AWS Access Key ID [None]: (Set Access Key ID)
	- AWS Secret Access Key [None]: (Set Secret Access Key)
	- Default region name [None]: (Set region to: us-east-1)
	- Default output format [None]: (Set Output format: json)

------------------------------------------------------------------------------------------------------------------------------

- INSTALL EB CLI IN JENKINS EC2 USER:

ADD TO PATH

Local path
```
export PATH='/var/lib/jenkins/.local/bin:$PATH')
```

Environment path
```
sudo nano /etc/environments
:~/.local/bin(add to the end of environment path)
```
Install AWS Elastic Beanstalk CLI 
```
sudo apt install python3-pip
sudo apt install python3.10-venv
pip install awsebcli --upgrade --user
eb --version
```
------------------------------------------------------------------------------------------

<h2>(2) Connect github to your Jenkins Server in order to test, build, and deploy the application.</h2>
- CONNECT GITHUB TO JENKINS SERVER:

	- First Fork the Deployment repo: (https://github.com/cceliuss187/kuralabs_deployment_2)
	- Next, create an access token from GitHub:
	- Navigate to your GitHub settings, select developer settings.
	- Select personal access token and create a new token.
	- Select the settings you see below for access token permissions
		- .repo
		- .admin:repo_hook
------------------------------------------------------------------------------------------



(7)
..........................................................................................
CREATE A MULTIBRANCH BUILD:
..........................................................................................
Log back into jenkins and select "New item".
Select multibranch pipeline.
enter a display name and brief description. 
Add a branch source by selecting Add source and select GitHub.
Select the add button and select GitHub.
Click on Add and then select Jenkins.
Under username enter your GitHub username.
Under password enter your token.
Enter your URL to the repository and you can validate by selecting validate.
Make sure this says Jenkinsfile.
Select Apply and then save.

You should see a build happening. if you don't, select Scan Repository.
------------------------------------------------------------------------------------------



(8)
..........................................................................................
NOW DEPLOY APPLICATION FROM ELASTIC BEANSTALK CLI:
..........................................................................................
sudo su - jenkins -s /bin/bash
cd /workspace/{{The name of your project}}/
eb init
	Select a default region: {{Select: us-east-1}}
	Select an application to use: {{Select: url-shortner_main}}
	It appears you are using Python. Is this correct? (Y/n): {{y}}
	Select a platform branch: {{Select the latest version of python available}}
	Do you wish to continue with CodeCommit? (Y/n): {{n}}
	Do you want to set up SSH for your instances? (Y/n): {{y}}
	Select a keypair: {{Select your keypair}}
eb create
	Enter Environment Name: {{press enter (default)}}
	Enter DNS CNAME prefix: {{press enter (default)}}
	Select a load balancer type: {{press enter (default)}}
	Would you like to enable Spot Fleet requests for this environment? (y/N): {{n}}
Wait for the environment to be made!! And then check it
-------------------------------------------------------------------------------------------



(9)
...........................................................................................
ADD A DEPLOYMENT STAGE TO THE PIPELINE IN YOUR JENKINSFILE:
...........................................................................................


stage ('deploy') {
       steps {
        sh '/var/lib/jenkins/.local/bin/eb deploy url-shortner-main-dev'
       }
     }
------------------------------------------------------------------------------------------




(10)
.......................................................................................
ADD A NEW TEST:
.......................................................................................

#Check to see if the response received is a status code of 404
def pgNOTfound():
    response = app.test_client().get('page_not_found.html')
    assert response.status_code == 404
---------------------------------------------------------------------------------------

- ISSUES

#sudo usermod -a -G sudo jenkins(add jenkins user to the "sudo" group)



	
	






