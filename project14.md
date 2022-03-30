CONTINOUS INTEGRATION WITH JENKINS, ANSIBLE, ARTIFACTORY SONARQUBE AND PHP
INTRODUTION
In this project, the concept of CI/CD is implemented whereby php application from github application are pushed to Jenkins to run a multi-branch pipeline job(build job is run on each branches of a repository simultaneously) which is better viewed from Blue Ocean plugin.  This is done in order to achieve continuous integration of codes from different developers. After which the artifacts from the build job is packaged and pushed to sonarqube server for testing before it is deployed to artifactory from which ansible job is triggered to deploy the application to production environment.
The following are the steps taken to achieve this:
STEP 0: Setting Up Servers
STEP 1:  Configuring Ansible For Jenkins Deployment
In order to run ansible commands from Jenkins UI, the following outlines the steps taken:
Installing Blue Ocean plugin from ‘manage plugins’ on Jenkins:
Creating new pipeline job on the Blue Ocean UI from Github
Generating new personal access token in order to full access to the repository
Pasting the token and selecting ansible-config-mgt repository to create a new pipeline job
Creating a directory called deploy where the Jenkinsfile will be created.
**The structure of the ansible-config-mgt**
Switching to new branch called ‘feature/Jenkinspipeline-stages’
Entering the following codes in the Jenkinsfile which does nothing but echo with shell script:
Committing the changes and heading over to the Jenkins console, clicking on ‘Configure’ to set the location of the Jenkinsfile.
Then clicking on ‘scan repository now’ will scan both the changes pushed to github and also the branches created in the repository and made them visible which means that this job is multibranch job which will trigger a build for each branch.
Clicking on ‘Build now’ to trigger the build
Installing Ansible plugin on Jenkins to be able to run ansible commands
Clearing the codes in the Jenkinsfile to start from the scratch
For ansible to be able to locate the roles in my playbook, the ansible.cfg is copied to /deploy folder and then exported from the Jenkinsfile code
Using the pipeline syntax Ansible tool to generate syntax for environment variable to set.
Introducing parameterization which enables us to input the appropriate values for the inventory file we want playbook to run against:
STEP 2: Setting Up The Artifactory Server
Since the goal here is to deploy applications directory from Artifactory rather than git, the following step is taken:
Creating account on the artifactory site
Creating the repository where the artifacts will be uploaded to from the Jenkins server
STEP 3: Integrating Artifactory Repository With Jenkins
Installing plot plugin to display tests reports and code coverage and Artifactory plugins to easily upload code artifacts into an Artifactory server
Configuring Artifactory in Jenkins on ‘configure systems’
Forking the repository into my Github account:
https://github.com/darey-devops/php-todo.git
On database server, installing mysql: `$ sudo yum install mysql-server`
Creating a database and a remote user:
Create database homestead;
CREATE USER 'homestead'@'<Jenkins-ip-address>' IDENTIFIED BY 'sePret^i';
GRANT ALL PRIVILEGES ON * . * TO 'homestead'@'<Jenkins-ip-address>';

On the Jenkins server, installing PHP, its dependencies
Installing Composer tool:
Updating the .env.sample with database server ip address
On VSCode, Creating Jenkinsfile for the php-todo repository on the main branch
Entering the following codes:
The first stage performs a clean-up which always deletes the previous workspace before running a new one
The second stage connects to the php-todo repository
The third stage performs a shell scripting which renames .env.sample file to .env, then composer is used by PHP to install all the dependent libraries used by the application and then php artisan uses the .env to setup the required database objects
Pushing the changes:
Creating php-todo pipeline job from the Blue Ocean UI
Running the pipeline job
After a successful run of the pipeline job, confirming on the database server by running SHOW TABLES command to see the tables being created
STEP 4: Structuring The Jenkinsfile
Including unit test stage in the Jenkinsfile:
Adding Code Quality stage with phploc tool and will save the output in build/logs/phploc.csv:
Adding the plot code coverage report stage:
Adding the package artifacts stage which archives the application code
Uploading the artifacts to the Artifactory repository in this stage:
Deploying the application to the dev environment by launching Ansible pipeline job(ansible-config-mgt)
STEP 5: Setting Up The SonarQube Server
To ensure that only code with the required code coverage and other quality standards gets to make it through to the dev environment, SonarQube is configured:
 On the SonarQube server, performing the following command on the terminal which makes the session changes which does not persist beyond the current session terminal to ensure optimal performance of the tool
sudo sysctl -w vm.max_map_count=262144
sudo sysctl -w fs.file-max=65536
ulimit -n 65536
ulimit -u 4096
To make the change permanent, editing limits.conf file:`$ sudo vi /etc/security/limits.conf`
Entering the following configuration:
sonarqube   -   nofile   65536
sonarqube   -   nproc    4096

Updating and upgrading the server: 
`$ sudo apt-get update`
`$ sudo apt-get upgrade`
Installing the wget and unzip packages: 
`$ sudo apt-get install wget unzip -y`

Installing OpenJDK and Java Runtime Environment(JRE) 11
` $ sudo apt-get install openjdk-11-jdk -y`
` $ sudo apt-get install openjdk-11-jre -y`
To install postgresql database, adding the postgresql repo to the repo list:
`$ sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" >> /etc/apt/sources.list.d/pgdg.list'`
Downloading software key:
`$ wget -q https://www.postgresql.org/media/keys/ACCC4CF8.asc -O - | sudo apt-key add -`
Installing postgresql database server:
`$ sudo apt-get -y install postgresql postgresql-contrib`
Starting postgresql server:
`$ sudo systemctl start postgresql`
Enabling it to start automatically:
`$ sudo systemctl enable postgresql`
Changing the password for the default postgres user:
`$ sudo passwd postgres`
Switching to the postgres user:
`$ su – postgres`
Creating a new user ‘sonar’:
`$ createuser sonar`
Activating postgresql shell:`$ psql`
Setting password for the newly created user for the SonarQube databases:
`ALTER USER sonar WITH ENCRYPTED password 'sonar';`
Creating a new database for Postgresql database:
`CREATE DATABASE sonarqube OWNER sonar;`
Granting all privileges to the user sonar on sonarqube database:
`grant all privileges on DATABASE sonarqube to sonar;`
Exiting from the shell
Switching back to sudo user:
To install SonarQube software, navigating to the /tmp folder to temporarily  download the installation files
`$ cd /tmp && sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-7.9.3.zip`
Unzipping the archive setup to the /opt directory:
`$ sudo unzip sonarqube-7.9.3.zip -d /opt`
Moving the extracted setup to /opt/sonarqube directory:
`$ sudo mv /opt/sonarqube-7.9.3 /opt/sonarqube`
Creating the group ‘sonar’:
`$ sudo groupadd sonar`
Adding a user with control over /opt/sonarqube directory:
`$ sudo useradd -c "user to run SonarQube" -d /opt/sonarqube -g sonar sonar `
sudo chown sonar:sonar /opt/sonarqube -R
