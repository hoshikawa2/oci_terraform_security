# How to integrate OCI Resource Manager with Automation Process

## Introduction

The Resource Manager service automates the deployment and operations of all Oracle Cloud Infrastructure resources. Using the infrastructure-as-code (IaC) model, the service is based on Terraform, an open source industry standard that allows DevOps engineers to develop and deploy their infrastructure anywhere.

A Terraform configuration encodes your infrastructure in declarative configuration files. The Resource Manager service allows you to share and manage infrastructure configurations and state files across multiple teams and platforms.


Through the OCI Resource Manager it is possible to execute Terraform scripts in the Oracle Cloud console, but it is also possible to make a REST call or through the OCI CLI, thus expanding the possibilities of integration with automation tools such as OCI Devops, Jenkins, github, among others.

![architecture](./images/CleanShot%202024-04-17%20at%2019.25.15.png)

Here let's show how to automate the deployment of an Oracle Cloud Autonomous Database instance by obtaining the database admin password securely through Oracle Cloud Vault Secrets without exposing it in the files and so that the Oracle Cloud Resource execution user Manager has proper access to create the database and stored password.

## Objectives

The objective of this material is to allow the configuration of automation through Terraform so that you can create instances with their appropriate access credentials but without exposing any sensitive information. Information such as password must be stored so that only authorized users can use it within the Terraform script.
The Terraform script will be executed through the OCI Resource Manager and the user must have permissions to:

- Create resources in the compartment
- Create an Autonomous Database instance
- Read an OCI Vault password
- Terraform script execution

## Pre Requirements

- Have an user inside an user group without any Policy. This user will be given the appropriate permissions to run Terraform in OCI Resource Manager
- An **OCI Object Storage** bucket created previously in a specific compartment (if you want to generate the **Terraform** script into this bucket)

## Task 1 - Create a Secret for Autonomous Database in OCI Vault

We will create a password in **OCI Vault / Secret** to illustrate how to configure a new resource in **Terraform** without expose a sensitive data.

**OCI Vault / Secrets** are credentials such as passwords, certificates, SSH keys, or authentication tokens that you use with Oracle Cloud Infrastructure services. Storing secrets in a vault provides greater security than you might achieve storing them elsewhere, such as in code or configuration files. You can retrieve secrets from the Vault service when you need them to access resources or other services.
You can create secrets by using the Console, CLI, or API. Secret contents for a secret are imported to the service from an external source. The Vault service stores secrets in vaults.

>**Note:** For this step, we need to log in with an Admin user in OCI. You must have permission to create a password in OCI Vault

Go to me Main menu and select **Identity & Security** and **Vault**

![img_10.png](images/img_10.png)

Select the compartment you want to store the secrets and click on **Create Vault** button

![img_29.png](images/img_29.png)

Put a name for your vault, confirm the compartment and click on **Create Vault** button

![img_11.png](images/img_11.png)
Confirm the vault creation and let's create a key. Click on **Create Key** button

![img_12.png](images/img_12.png)

Confirm the compartment and put a name for your key. For example, put **autonomouskey** on Name and click on **Create Key** button

![img_13.png](images/img_13.png)
Confirm the key creation

![img_14.png](images/img_14.png)
Now, click on **Secrets** option and click on **Create Secret** button

![img_15.png](images/img_15.png)

We will create the **Autonomous Admin** password. Confirm the compartment, put a name for your first secret. Select **Manual secret generation** to include the password. Select **Plain-Text** option on **Secret Type Template** and write your password. Finally, click on **Create Secret** button.

![img_16.png](images/img_16.png)

You will need the **Secret OCID**. Copy the **OCID** value from the **Secret** detail page

![img_17.png](images/img_17.png)

## Task 2 - Create Policies to the OCI user

In this tutorial, consider your username **TestUser** included in a group named **TestGroup**. Let's create a **Policy** with name **TestPolicy**.

This step is very important because here is the control of all privileges needed to guarantee the security of **Terraform's** automation.

>**Note:** For this step, we need to log in with an Admin user in OCI.

![img.png](images/img.png)

Go to the Main Menu, select **Identity & Security** and select **Policies**.

![img_18.png](images/img_18.png)

Click in **Create Policy** button

![img_19.png](images/img_19.png)

And include the policies:

![cloud_shell.png](images/cloud_shell.png)

    This policy gives the permission to group TestGroup created previously to manage a Stack and jobs in Resource Manager
    - Allow group 'Default'/'TestGroup' to manage orm-stacks in compartment integration
    - Allow group 'Default'/'TestGroup' to manage orm-jobs in compartment integration
    - Allow group 'Default'/'TestGroup' to read orm-config-source-providers in tenancy

    This policy gives the right to create an Autonomous Database instance in the compartment integration
    - Allow group 'Default'/'TestGroup' to manage autonomous-database in compartment integration

    The group can read the password stored in Vault/Secret through Terraform scripts 
    - Allow group 'Default'/'TestGroup' to use secret-family in tenancy

    This policy gives the right to save the Terraform scripts on a specific compartment
    - Allow group 'Default'/'TestGroup' to manage all-resources in compartment kubernetes

    This policy allows the users of TestGroup to edit code in **OCI Code Editor**
    - Allow group 'Default'/'TestGroup' to use cloud-shell in tenancy

## Task 3 - Create a Stack from a Template

Let's create a stack for an Autonomous Database instance. We can use a template for this.
The first step is log in as the user created previously.

Now, select the Main Menu

![img_1.png](images/img_1.png)

And go to **Developer Services** and **Resource Manager** / **Stacks**

![img_2.png](images/img_2.png)

Select your **compartment** and click on **Create stack** button

![CleanShot 2024-04-15 at 07.47.32.png](images/CleanShot%202024-04-15%20at%2007.47.32.png)

Now select **Template** option and click on **Select template** button to choose to generate a **Terraform** script for the the **Autonomous Database** 

![CleanShot 2024-04-15 at 07.48.19.png](images/CleanShot%202024-04-15%20at%2007.48.19.png)

In **Service** tab, find the **Autonomous Transaction Processing Database** option and click to mark this option and then click on **Select template** button 

![CleanShot 2024-04-15 at 07.49.57.png](images/CleanShot%202024-04-15%20at%2007.49.57.png)

You can generate the **Terraform** scripts and store into a **OCI Object Storage** bucket.
Select **Use custom Terraform providers**, then choose the bucket **compartment** and **name**.
Save your **stack**

![CleanShot 2024-04-15 at 07.51.23.png](images/CleanShot%202024-04-15%20at%2007.51.23.png)

You stack is saved 

![CleanShot 2024-04-15 at 07.54.01.png](images/CleanShot%202024-04-15%20at%2007.54.01.png)

This template does not read the secret stored in your **OCI Vault**. To make the **Terraform** to read the **secret**, we need to change the code.

Click on **Edit** select box and choose the **Edit Terraform configuration in code editor** option.

![img_3.png](images/img_3.png)

You can now edit the code. The default code generates a random string for the password

**Figure 1 - main.tf** file
![CleanShot 2024-04-15 at 08.05.40.png](images/CleanShot%202024-04-15%20at%2008.05.40.png)

**Figure 2 - main.tf** file
![CleanShot 2024-04-15 at 08.08.32.png](images/CleanShot%202024-04-15%20at%2008.08.32.png)

You need to add a new data named **oci_secrets_secretbundle** and assign it to the attributes:

- **admin_password** at autonomous_data_warehouse and autonomous_database positions
- **password** at autonomous_database_wallet position

**Figure 3 - main.tf / autonomous_data_warehouse positon**
![CleanShot 2024-04-15 at 08.28.07.png](images/CleanShot%202024-04-15%20at%2008.28.07.png)

    data "oci_secrets_secretbundle" "bundle" {
        secret_id = var.secret_ocid
    }

    admin_password = base64decode(data.oci_secrets_secretbundle.bundle.secret_bundle_content.0.content)

**Figure 4 - main.tf / autonomous_database position**  
![img_4.png](images/img_4.png)

**Figure 5 - main.tf / autonomous_database_wallet position**
![img_5.png](images/img_5.png)

![CleanShot 2024-04-15 at 08.30.15 substituir.png](images/CleanShot%202024-04-15%20at%2008.30.15%20substituir.png)

Include this code into **variables.tf** file and replace the **OCID** for your secret generated previously


      variable "secret_ocid" {
        default = "ocid1.vaultsecret.oc1.iad.xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
      }

**Figure 6 - variables.tf** file
![CleanShot 2024-04-15 at 08.30.52.png](images/CleanShot%202024-04-15%20at%2008.30.52.png)

>**Note:** Save your files now. Move the mouse cursor over your Stack (on the right side of the editor, in "Autonomous Transaction Processing...." title), click the right button and save your project. If you get out without saving, the execution assumes you will use the random string in the original code script.

## Task 4 - Test permissions

You can test the **Policies** and see how you have control of **OCI Resource Manager**, **OCI Vault / Secret** and **Autonomous** instances in a specific **compartment**.

Let's do some tests. First, with your **Admin user**, log in into **OCI** and remove all the **Policies** for the group **TestGroup** in the **TestPolicy** Policy. Click on **Delete** button and confirm.

![img_7.png](images/img_7.png)

Now, log in with your **user** in the group **TestGroup** and you cannot see the stack, so you cannot execute it.

![CleanShot 2024-04-15 at 20.02.33.png](images/CleanShot%202024-04-15%20at%2020.02.33.png)

Now, with the **Admin user**, include this statement:

    Allow group 'Default'/'TestGroup' to manage orm-stacks in compartment integration
    Allow group 'Default'/'TestGroup' to manage orm-jobs in compartment integration
    Allow group 'Default'/'TestGroup' to read orm-config-source-providers in tenancy
    Allow group 'Default'/'TestGroup' to manage all-resources in compartment kubernetes

![img_8.png](images/img_8.png)

This statements grant your **user** in **TestGroup** to use the **OCI Resource Manager** stack.

![CleanShot 2024-04-15 at 20.12.55.png](images/CleanShot%202024-04-15%20at%2020.12.55.png)

We removed the grant for your **user** to create an **Autonomous** instance and read the **secret** in **OCI Vault**. So you can execute your **stack** but with no success. To test, click on **apply** button in your **stack** detail page.

![CleanShot 2024-04-15 at 20.16.39.png](images/CleanShot%202024-04-15%20at%2020.16.39.png)
![CleanShot 2024-04-15 at 20.18.40.png](images/CleanShot%202024-04-15%20at%2020.18.40.png)

Now, let's include **Autonomous** and **OCI Vault** permissions on the **TestPolicy**

    Allow group 'Default'/'TestGroup' to manage all-resources in compartment kubernetes
    Allow group 'Default'/'TestGroup' to manage autonomous-database in compartment integration

![img_9.png](images/img_9.png)

Click again on **apply** button in your **stack** detail page and you can see you have the control about all resources without exposing any password.

![CleanShot 2024-04-15 at 21.34.57.png](images/CleanShot%202024-04-15%20at%2021.34.57.png)

## Task 5 - Call your OCI Resource Manager automation with REST 

All the resources in **OCI** have a **REST API** or an **OCI CLI** command to call services as is executed in your **OCI** console.
If you don't know about **OCI REST API**, you can see here: [OCI REST API](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/usingapi.htm). You can view this list os **OCI CLI** commands here: [OCI CLI](https://docs.oracle.com/en-us/iaas/tools/oci-cli/3.39.0/oci_cli_docs/)

Now we can choose to execute the **OCI Resource Manager** stack with a **REST** ou an **OCI CLI** command.

You can create a **Job** with this material: [Create a Job](https://docs.oracle.com/en-us/iaas/api/#/en/resourcemanager/20180917/Job/CreateJob) or you can use this sample code [curl-oci.zip](./files/curl-oci.zip) with **curl-oci.sh** prepared with **OCI** parameters.

![img_20.png](images/img_20.png)

You will need to create the signature for your **REST** requests.
[Oracle Cloud Infrastructure (OCI) REST call walkthrough with curl](https://www.ateam-oracle.com/post/oracle-cloud-infrastructure-oci-rest-call-walkthrough-with-curl)


First, create a file named **STACK-RUN.sh**. This will be your REST request using a **curl-oci.sh** (this tool will prepare your authorization string with your **OCI** information).

![img_1.png](images/rest_api_2.png)

The next step is create a file named **request.json** with your **stack ID** and **compartment ID** reference. 

![img.png](images/rest_api_1.png)

Open the **curl-oci.sh** file and change these parameters. These parameters are the same of your **OCI CLI** installation.

![img_2.png](images/rest_api_3.png)

And now you can execute the script

![img_3.png](images/rest_api_4.png)

You can see the success results

![img_4.png](images/rest_api_5.png)

## Acknowledgments

- Author: Cristiano Hoshikawa (Oracle LAD A-Team Solution Engineer)

## References

- [Using OCI Vault Secrets for Terraform resources](https://blogs.oracle.com/developers/post/using-oci-vault-secrets-for-terraform-resources)
- [Securing Resource Manager](https://docs.oracle.com/en-us/iaas/Content/Security/Reference/resourcemanager_security.htm)
- [Overview of IAM](https://docs.oracle.com/en-us/iaas/Content/Identity/getstarted/identity-domains.htm#identity_documentation)
- [Create a Job](https://docs.oracle.com/en-us/iaas/api/#/en/resourcemanager/20180917/Job/CreateJob)
- [OCI REST API](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/usingapi.htm)
- [OCI CLI](https://docs.oracle.com/en-us/iaas/tools/oci-cli/3.39.0/oci_cli_docs/)
- [Oracle Cloud Infrastructure (OCI) REST call walkthrough with curl](https://www.ateam-oracle.com/post/oracle-cloud-infrastructure-oci-rest-call-walkthrough-with-curl)
- [Managing Autonomous Data Warehouse Using oci-curl](https://blogs.oracle.com/datawarehousing/post/managing-autonomous-data-warehouse-using-oci-curl)
