ANSIBLE REFACTORING AND STATIC ASSIGNMENT (IMPORTS AND ROLES)
INTRODUCTION
In continuation of project 11, the ansible code in my ansible-config-mgt repository is refactored by making use of the import functionality and assigning task in the playbook with role functionality.
The following outlines the steps taken:
STEP 1: Enhancing The Jenkins Job
Since artifacts stored in Jenkins server changes directory at each build, copy-artifacts plugin is being used to copy artifacts anytime a build is triggered and paste it automatically to a specified directory where ansible playbook can be run with ease.
Creating a new directory on the Jenkins-ansible server where the artifacts will be copied to:`$ sudo mkdir /home/ubuntu/ansible-config-artifact`
Changing the permissions:
`$ chmod -R 0777 /home/ubuntu/ansible-config-artifact`
On the Jenkins web console, onto the manage plugin, on the available tab and searching for copy-artifact and installing the plugin
Creating a new freestyle job ‘save-artifact’ and configuring it to be triggered on the completion of the existing ‘ansible’ job:
**On the general tab**
**On the source code management**
**On the build tab**
Testing the configuration by making a change in the README.md file
**ansible build triggered**
**save-artifact job triggerd**
**artifacts successfully saved on the /home/ubuntu/ansible-config-artifact**
STEP 2: Refactoring Ansible Code
Pulling the latest changes from the main branch on VSCode
Checkout to new branch ‘refactor’: `git checkout -b refactor`
Creating site.yml file in the playbooks folder. The file will be considered as an entry point into the entire infrastructure, ie, site.yml will be the parent to all other playbook
Creating ‘static-assignments’ folder in the root of the repository. This is where all the children playbooks are stored 
Moving the common.yml file to the static-assignments folder
Configuring the site.yml file to import common.yml file
**The ansible-config-artifact folder structure**
Since the wireshark has been installed on the webservers, so common-del.yml is used instead of common.yml to uninstall wireshark

**common-del.yml playbook file**
**site.yml playbook file**

Running the ansible-playbook command against dev.yml inventory file
`$ sudo ansible-playbook -i /home/ubuntu/ansible-config-artifact/inventory/dev.yml /home/ubuntu/ansible-config-artifact/playbooks/site.yml

STEP 3: Making Use Of Role Functionalities
To demonstrate the role functions, 2 new Red Hat servers are launched and are used as uat.
Configuring the uat.yml in the inventory folder with the ip address of the 2 new servers launched
To create a role for the uat, the folder ‘role’ is created in the playbooks directory
Creating the role structure ‘webserver’ with Ansible utility: `$ sudo ansible-galaxy init webserver`
**webserver role folder structure**
Inorder to make ansible locate the role directory, uncomment the role section and specify the role path in the ansible.cfg file
Configuring the task file of the webserver role and adding the following task: `$ sudo vi ansible-config-artifact/playbooks/role/webserver/task/main.yml`
Install and configure Apache (httpd service)
Clone Tooling website from GitHub https://github.com/<your-name>/tooling.git.
Ensure the tooling website code is deployed to /var/www/html on each of 2 UAT Web servers.
Make sure httpd service is started

Creating a new assignment in the static-assignments folder ‘uat-webservers’: `$ sudo vi ansible-config-artifact/static-assignments/uat-webservers.yml`
Entering the following codes:
Updating the site.yml file to be able to import uat-webservers role
STEP 4: Commit And Running the playbook
Commiting the changes and pushing the code to the main branch to create a merge request
Merging the request
Ensuring that Jenkins build is triggered
Running the ansible playbook
