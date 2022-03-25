ANSIBLE DYNAMIC ASSIGNMENTS(INCLUDE) AND COMMUNITY ROLES

INTRODUCTION
In continuation with project 12, dynamic assignment is introduced by making use of include modules. By dynamic, it means that all statements are processed only during execution of the playbook which is the opposite of the import modules.
The following steps outlines how include module is used for running dynamic environment variable:
STEP 1: Introducing Dynamic Assignment Into Our Structure
Checking out to a new branch in the same ansible-config-mgt repository and naming it ‘dynamic-assignments’
Creating a new folder in the root directory of the repository and naming it ‘dynamic-assignments’
Creating an environment variable file in the dynamic-assignments directory and naming it ‘env_vars.yml’
Creating a folder that holds the env-var files and naming it ‘env-var’
Creating the following files under it: dev.yml, uat.yml, prod.yml and stage.yml
The structure of the ansible-config-mgt folder will be as displayed below:
Entering the following codes in the env_vars.yml file:
Updating site.yml file to work with dynamic-assignments:
STEP 2: Implementing Community Roles
In order to preserve my github state after installing a new role by committing the changes made and pushing it to the ansible-config-mgt repo directly from the bastion server, I installed git packages: `$ sudo apt install git`
Initializing the ansible-artifact-config directory: `$ git init`
Pulling the ansible-config-mgt repository:
git pull https://github.com/<your-name>/ansible-config-mgt.git
registering the repo 
git remote add origin https://github.com/<your-name>/ansible-config-mgt.git
Creating a new branch:
git branch roles-feature
switching to the new branch
git switch roles-feature
installing a MySQL role already configured from ansible-galaxy by geerlingguy in the role directory
ansible-galaxy install geerlingguy.mysql
renaming the role to mysql:
mv geerlingguy.mysql/ mysql
git add .
git commit -m "Commit new role files into GitHub"
git push --set-upstream origin roles-feature

Creating a pull request

Merging the request

STEP 3: Implementing Load Balancer Roles
Two load balancer roles are setup which are Nginx and Apache roles, but because a server can only make use of one, the playbook is configured with the use of conditionals- when statement, to ensure that only the desired load balancer role tasks gets to run on the webserver 
Setting up apache role: `$ sudo ansible-galaxy init apache`
Setting up nginx role: `$ sudo ansible-galaxy init nginx`
Entering the following code in apache/tasks/main.py
Entering the following code in nginx/tasks/main.py
Declaring the following variable in the defaults/main.py in both load balancer roles.

apache/defaults/main.py
enable_nginx_lb: false
load_balancer_is_required: false
nginx/defaults/main.py
enable_apache_lb: false
load_balancer_is_required: false

Creating a file in the static-assignment folder and naming it ‘loadbalancer.yml’
Entering the following codes:
- hosts: lb
  roles:
    - { role: nginx, when: enable_nginx_lb and load_balancer_is_required }
    - { role: apache, when: enable_apache_lb and load_balancer_is_required }

Updating the site.yml file:

To define which load balancer to use, the files in the env-var folder is used. In this case the env-var/dev.yml file is used to define nginx load balancer:

enable_nginx_lb: true
load_balancer_is_required: true
Running the playbook
