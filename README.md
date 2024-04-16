# How to integrate OCI Resource Manager with Automation Process

## Introduction

The Resource Manager service automates the deployment and operations of all Oracle Cloud Infrastructure resources. Using the infrastructure-as-code (IaC) model, the service is based on Terraform, an open source industry standard that allows DevOps engineers to develop and deploy their infrastructure anywhere.

A Terraform configuration encodes your infrastructure in declarative configuration files. The Resource Manager service allows you to share and manage infrastructure configurations and state files across multiple teams and platforms.


Through the OCI Resource Manager it is possible to execute Terraform scripts in the Oracle Cloud console, but it is also possible to make a REST call or through the OCI CLI, thus expanding the possibilities of integration with automation tools such as OCI Devops, Jenkins, github, among others.

Here I will show how to automate the deployment of an Oracle Cloud Autonomous Database instance by obtaining the database admin password securely through Oracle Cloud Vault Secrets without exposing it in the files and so that the Oracle Cloud Resource execution user Manager has proper access to create the database and stored password.

## Objectives

The objective of this material is to allow the configuration of automation through Terraform so that you can create instances with their appropriate access credentials but without exposing any sensitive information. Information such as password must be stored so that only authorized users can use it within the Terraform script.
The Terraform script will be executed through the OCI Resource Manager and the user must have permissions to:

- Create resources in the compartment
- Create an Autonomous Database instance
- Read an OCI Vault password
- Terraform script execution

## Pre Requirements

- Have a user and a user group. This user will be given the appropriate permissions to run Terraform in OCI Resource Manager



![CleanShot 2024-04-15 at 07.47.32.png](CleanShot%202024-04-15%20at%2007.47.32.png)
![CleanShot 2024-04-15 at 07.48.19.png](CleanShot%202024-04-15%20at%2007.48.19.png)
![CleanShot 2024-04-15 at 07.49.57.png](CleanShot%202024-04-15%20at%2007.49.57.png)
![CleanShot 2024-04-15 at 07.51.23.png](CleanShot%202024-04-15%20at%2007.51.23.png)
![CleanShot 2024-04-15 at 07.54.01.png](CleanShot%202024-04-15%20at%2007.54.01.png)
![CleanShot 2024-04-15 at 08.05.40.png](CleanShot%202024-04-15%20at%2008.05.40.png)
![CleanShot 2024-04-15 at 08.08.32.png](CleanShot%202024-04-15%20at%2008.08.32.png)
![CleanShot 2024-04-15 at 08.28.07.png](CleanShot%202024-04-15%20at%2008.28.07.png)
![CleanShot 2024-04-15 at 08.30.15 substituir.png](CleanShot%202024-04-15%20at%2008.30.15%20substituir.png)
![CleanShot 2024-04-15 at 08.30.52.png](CleanShot%202024-04-15%20at%2008.30.52.png)
![CleanShot 2024-04-15 at 20.02.33.png](CleanShot%202024-04-15%20at%2020.02.33.png)
![CleanShot 2024-04-15 at 20.12.37.png](CleanShot%202024-04-15%20at%2020.12.37.png)
![CleanShot 2024-04-15 at 20.12.55.png](CleanShot%202024-04-15%20at%2020.12.55.png)
![CleanShot 2024-04-15 at 20.16.39.png](CleanShot%202024-04-15%20at%2020.16.39.png)
![CleanShot 2024-04-15 at 20.18.40.png](CleanShot%202024-04-15%20at%2020.18.40.png)
![CleanShot 2024-04-15 at 20.26.37.png](CleanShot%202024-04-15%20at%2020.26.37.png)
![CleanShot 2024-04-15 at 21.26.11.png](CleanShot%202024-04-15%20at%2021.26.11.png)
![CleanShot 2024-04-15 at 21.34.57.png](CleanShot%202024-04-15%20at%2021.34.57.png)