# AUTOMATING INFRASTRUCTURE WITH IAC USING TERRAFORM PART 4 – TERRAFORM CLOUD
## INTRODUCTION
In this project, instead of running the Terraform codes in project 18 from a command line, rather it is being executed from Terraform cloud console. The AMI is built different with packer while Ansible is used to configure the infrastructure after its been provisioned by Terraform.

The following outlines the steps:

## STEP 1: Setting Up A Terraform Account

- After verifying my email and creating an organisation in the terraform cloud site, then on the **configure workspace** page, selecting **version control workflow** option inorder to run Terraform commands triggered from my git repository.
- Creating a new repository called terraform-cloud and pushing my terraform codes developed in project 18 into the repository
- Connecting the workspace to the new repository created and clicking on **create workspace**

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project19/10-creating%20a%20workspace.png)
![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project19/10-creating%20workspace-2.png)
![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project19/10-creating%20workspace-3.png)

- Clicking on **configure variables** on the next page to setup my AWS credentials as environment variables.

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project19/11-creating%20variables.png)

- Now my Terraform cloud is all set to apply the codes from GitHub and create all necessary AWS resources.

## STEP 2: Building AMI With Packer

- Installing **packer** on my local machine:

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project19/install%20packer.png)

- Cloning the [repository](https://github.com/darey-devops/PBL-project-19.git) and changing directory to the AMI folder

- Running the packer commands to build AMI for Bastion server, Nginx server and webserver

**For Bastion Server**

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project19/packer%20build%20bastion.png)
![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project19/packer%20build%20bastion-2.png)

**For Nginx Server**

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project19/packer%20build%20nginx.png)
![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project19/packer%20build%20nginx-2.png)

**For Tooling and Wordpress Server**

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project19/packer%20build%20web.png)
![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project19/packer%20build%20web-2.png)

**For Jenkins, Artifactory and sonarqube Server**

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project19/packer%20build%20ubuntu.png)
![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project19/packer%20build%20ubuntu-2.png)

## STEP 3: Running The Terraform Cloud To Provision Resources

- Inputing the AMI ID in my **terraform.tfvars** file for the servers built with packer which terraform will use to provision Bastion, Nginx, Tooling and Wordpress server

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project19/updating%20ami%20details.png)

- Pushing the codes to my repository will cause terraform cloud to trigger a plan
- Accepting the plan to to trigger an apply command

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project19/terraform%20apply-1.png)
![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project19/terraform%20apply-2.png)
![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project19/terraform%20apply-3.png)

## STEP 4: Configuring The Infrastructure With Ansible

- After a successful execution of terraform apply, connecting to the bastion server through ssh-agent to run ansible against the infrastructure

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project19/connecting%20to%20the%20bastion.png)

- Updating the **nginx.conf.j2** file to input the internal load balancer dns name generated via terraform:

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project19/updating%20nginx%20conf.png)

- Updating the RDS endpoints, Database name, password and username in the **setup-db.yml** file for both the tooling and wordpress role

**For Tooling**

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project19/updating%20setup-db%20for%20tooling.png)

**For Wordpress**

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project19/updating%20setup-db%20for%20wordpress.png)

- Updating the EFS Access point ID for both the wordpress and tooling role in the **main.yml**

**For Tooling**

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project19/updating%20the%20efs%20point%20of%20tooling.png)

**For Wordpress**

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project19/updating%20the%20efs%20point%20of%20wordpress.png)

- Verifying the inventory

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project19/ansible%20inventory%20graph.png)

- Exporting the environment variable **ANSIBLE_CONFIG** to point to the ansible.cfg from the repo and running the ansible-playbook command: `$ ansible-playbook -i inventory/aws_ec2.yml playbook/site.yml`

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project19/running%20ansible%20playbook.png)
![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project19/running%20ansible%20playbook-2.png)

## STEP 5: Setting Slack Notification For trigger Events

- From the notification option in the settings tab, selecting slack option

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project19/setting%20up%20slack%20notification-1.png)
![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project19/setting%20up%20slack%20notification-2.png)

- Creating the webhook to use to setup the notification

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project19/setting%20up%20slack%20notification-3.png)

## STEP 6: Setting The Terraform Cloud To Execute Only From 'Dev' Branch

- Creating 3 branches dev, prod and test 

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project19/3%20branches%20created.png)

- On the version control page in the settings tab, setting the **VCS branch** to **dev**

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project19/running%20trigger%20from%20dev%20branch.png)

- After pushing the code to the dev branch the terraform plan command is triggered and notifications received from slack:

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project19/notifications%20received.png)

- Destroying the infrastructure by clicking on **Destruction and Delete** option in the settings tab:

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project19/destroying%20the%20infrastructure.png)
![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project19/destroying%20the%20infrastructure-2.png)

## STEP 7: Working With Terraform Private Module Registry

- Forking the repo from [hashicorp](https://github.com/hashicorp/learn-private-module-aws-s3-webapp)
- Then under my repository's tab, clicking on **tag** to create tag, clicking 'Create a new release' and adding 1.0.0 to the tag version field setting the release title to "First module release"

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project19/creating%20release%20tag.png)
![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project19/creating%20release%20tags-2.png)

- Clicking **Publish release** to create the release
- To create a Terraform module for my private module registry, navigating to the Registry header in Terraform Cloud and selecting **Publish private module** from the upper right corner.
- Selecting the GitHub(Custom) VCS provider that I configured and choosing the name of the module repository **terraform-aws-s3-webapp** and clicking the **Publish module** button.

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project19/signing%20in%20to%20terraform%20registry.png)
![]()
- To create a configuration that uses the module; forking the repo [learn private module](https://github.com/hashicorp/learn-private-module-root/) which will access the module I created and Terraform will use it to create infrastructure.

**main.tf**
**variables.tf**
**outputs.tf**
- Creating a new workspace and selecting the **learn-private-module-root** repo 
- Clicking on **Configure Variable** to set my AWS credentials as environment variable and also set the values of these variables: region, prefix and name which is specified in the root module configuration
- Deploying the infrastructure by clicking on **start new plan**

**Testing the Infrastructure**

- Destroying the Infrastructure
