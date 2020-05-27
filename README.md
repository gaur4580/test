[[_TOC_]]

# Introduction  

Current version of the VM Jumpstart Kit is v0.3.1 compatible with common-library v0.3.1

This repository is one in a series of [JumpStart Kits](https://dev.azure.com/ATTDevOps/ATT%20Cloud/_wiki/wikis/DevOps%20Automation/437/JumpStart-Kits). This kit will highlight a particular combination of patterns at the [Compute and Persistence layer](https://dev.azure.com/ATTDevOps/ATT%20Cloud/_wiki/wikis/DevOps%20Automation/437/JumpStart-Kits?anchor=purpose). The Compute layer of this kit will use [Azure Virtual Machine](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/overview) (VM) for the middle tier. The frontend tier will also use VM, but can be swapped using a different Azure Service like Azure IaaS VMs. The Persistence Layer of this kit will use Azure PaaS DB for the backend database tier.

> Note: Please ensure you read through the major [JumpStart Kits](https://dev.azure.com/ATTDevOps/ATT%20Cloud/_wiki/wikis/DevOps%20Automation/437/JumpStart-Kits) documentation before continuing with this doc.

# Example Use Case

This kit will deploy the following target architecture.

A. Compute Layer

1. The Front-end Tier provisions a Virtual Machine running [Springboot sample app](TODO create Springboot sample app)

B. Persistence Layer

3. Backend Tier will have [Azure database for MySQL DB](TODO create Azure database for MySQL) via Azure PaaS DB. <!-- TODO POC Azure Cosmos DB -->

The below diagram is a visual representation of this target architecture.

## Target Architecture Diagram

![image.png](/.attachments/VM-with-PaaS.png)

# Getting Started

If you find that this JumpStart Kit will be useful for accelerating your migration strategy, please follow the steps found at the wiki, [Getting Started on a kit](https://dev.azure.com/ATTDevOps/ATT%20Cloud/_wiki/wikis/DevOps%20Automation/435/How-To-Consume).

## High-level steps for consuming kit

1. Copy and run the [DevOps Services](https://dev.azure.com/ATTDevOps/ATT%20Cloud/_git/att-onboarding-devops-services) automation.
2. Create a new repo in the VP tower for the application
3. Copy this kit repo to the new repo in the VP tower into a new branch
4. Create Branch from a specific version of the kit compatible with the desired common-library version (this will be your working branch)  
    `git checkout v0.1.0 -b newbranch`
5. Modify /Pipelines/Pipeline.yaml to reference desired common-library version
6. Modify /Pipelines/Variables.yaml to suit your needs
7. Modify /Pipelines/Variables.<environment>.yaml for each environment (Service Principals used for each environment)
8. Create an Azure Pipeline pointing to Pipelines/Pipeline.yaml
9. Modify /Terraform/Backends/backend.<environment>.tfvars for each environment
10. Modify /Terraform/Config/config.<environment>.tfvars to declare the Azure resource properties (names, vnet info, etc...)

## Detailed steps for consuming kit

### Pre-Requisites

Before running this Jumpstart Kit for the first time there are some required resources needed.  

- Run the [DevOps Services Automation](https://dev.azure.com/ATTDevOps/ATT%20Cloud/_git/att-onboarding-devops-services). This is the repository that deploys the base infrastructure in the application subscription. This will create some of resources relevant to this kit including: Vnet, Subnets, Storage Account, Shared Image Gallery, Keyvault.
- Set up and configure a self hosted custom agent pool.

If planning to use Image Builder piece of this kit then the following steps are required after provisioning the DevOps Services and creating the custom agent pool:

- The custom agent pool must be deployed to the same VNET used by Packer.
- Access to the ATT Golden Images Shared Image Gallery (TODO: add instructions on how to request access)
- Verify the Service Connection has "Storage Blob Data Reader" to the Storage Account.
- Verify the SPN has "Contributor" Role to the destination Shared Image Gallery.
- Manually upload the sample "TODOAPP" to a storage account container. More details [here](#image-builder-required-resources)

### Pipeline Preparation

#### Create a new repo in the VP tower for the application

An existing VP tower and the ability to create a new repo should already be in place

#### Copy this kit repo to the new repo in the VP tower

Within the new repo, create a branch while code is being adjusted to a working state. After confirming that this kit is appropriate for the application, either clone or copy it from this location to the new repo. This will become your working copy.

#### Modify Pipelines/Pipeline.yaml

The provided `/Pipelines/Pipeline.yaml` file contains three pre-defined stages with dependencies. Each stage is considered an environment which must have its own corresponding ./Terraform/Config/config.<environment>.tfvars file. If a new environment is needed, copy and paste a stage and modify the name. This also supports same environment multi-region. For example a multi-region deployment of DEV would have two stages. One named DEV-EASTUS and the other as DEV-EASTUS2. There would be two TFVARS files named config.dev-eastus.tfvar and config.dev-eastus2.tfvar. See "Terraform Prep".

#### Set Common Library version

The common library includes Terraform modules that are common across projects and towers. It also includes Pipeline YAML sequences that are common across all pipelines, which are the Terraform setup, init, plan, and apply stages. These are referenced and versioned in the "resources" section of the pipeline.yaml file. **Ensure the version tag is used as there is ongoing development being merged regularly with master.**  

```yml
resources:
  repositories:
  - repository: common-library
    type: git
    name: ATT Cloud/common-library
    ref: refs/tags/v0.1 # can also be a branch reference if required
    endpoint:
```
  
#### Modify Pipeline/Variables.yaml

Edit the example /Pipelines/Variables.yaml file, and identify:

1. MOTSID = Application identifier. Used for creation of resource group and state file
2. poolName = In most cases this can be left as "Custom" which will use the ADO agents already provisioned in your tower
3. Image Builder variables. Please review .VARIABLE* comments at the top of the document `Pipelines/Variables.yaml` to understand each variable.

#### Create an Azure Pipeline  

In Azure DevOps Pipelines, navigate to Pipelines -> New -> Existing YAML -> point to /Pipelines/Pipeline.yaml  

### Terraform Preparation

#### Modify Terraform Backends Files

Create or edit a /Terraform/Config/backend.<ENVIRONMENT>.tfvars file for each environment. This contains the Terraform state configuration for the backend

### Setup Terraform Variables Per Environment

Create or edit a /Terraform/Config/config.<ENVIRONMENT>.tfvars file for all variable definitions per environment. Three sample environments have been provided by default (DEV, QA, PROD)

### Test the deployment

At this point you should be able to commit your changes to the new repo and branch, the pipeline will automatically run if the CI triggers are set otherwise run it manually.

# Image Builder

Image Builder is the pipeline job that allows you to create a managed image from a base or 'Golden' image. When a CI artifact from a storage account is specified, Packer is then used to create an immutable image that can be published to a Shared Image Gallery.  

Please review .VARIABLE* comments at the top of the document `Pipelines/Variables.yaml` to understand each variable.

## Image Builder Required Resources

The [DevOps Services Automation](https://dev.azure.com/ATTDevOps/ATT%20Cloud/_git/att-onboarding-devops-services) includes the following resources that should already exist before running the pipeline. Image builder expects the following shared resources to exist on the current subscription prior to running the pipeline:

**.VARIABLE dest_sig_rg**  
*The destination shared image gallery resource group in DevOps services*  
**.VARIABLE dest_sig**  
*The destination shared image gallery in DevOps services*  
**.VARIABLE vp_base_rg**  
*The base resource group in DevOps services. This rg will contain the virtual network*  
**.VARIABLE ci_sa_name**  
*The ci storage account name in DevOps services*  

> Note:
> The service connection used to access the 'ci_sa_name' storage account must have the "Storage Blob Data Reader" reader to the CI Storage Account created by the DevOps Services Automation. Instructions to add RBAC roles for access to blob storage [here](https://docs.microsoft.com/en-us/azure/storage/common/storage-auth-aad-rbac-portal)

**.VARIABLE todoapp_container_name**  
*The name of the container that hosts TodoApp's blob*  
**.VARIABLE todoapp_blob**  
*The name of the blob that hosts TodoApp's zip file*  

> Note:
> Manually create a container on the "ci_sa_name" storage account and upload the "TODOAPP" zip file to the container, this is a sample application that image builder will use to create the image.

![image.png](/.attachments/BlobContainerAndBlob.PNG)

The following variables are part of the Packer build, if you are using a self hosted custom agent then it must be deployed on the same RG, vnet and subnet declared below.

**.VARIABLE vp_packer_rg**  
*The resource group to place the custom image in DevOps services*  
**.VARIABLE vp_vnet**  
*The virtual network in DevOps services*  
**.VARIABLE vp_packer_subnet**  
*The subnet in DevOps services to deploy the temporary packer vm provisioner*  

The base Shared Image Gallery is where the base or "Golden" images are located. ATT has a Shared Image Gallery with the approved golden images, access to this resource is up to the application teams. However, you can create these resources manually for initial testing of the kit. For addition instructions on how to create a SIG, a definition and an image version: [https://docs.microsoft.com/en-us/azure/virtual-machines/windows/shared-image-galleries](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/shared-image-galleries)  

**.VARIABLE base_sig**  
*The base shared image gallery name where the vanilla image exists*  
**.VARIABLE base_sig_rg**  
*The base shared image gallery resource group name where the vanilla image exists*  
**.VARIABLE base_sig_subid**  
*The base shared image gallery's subscription id where the vanilla image exists*  
**.VARIABLE base_sig_definition**  
*The base shared image definition where the vanilla image exists*  
**VARIABLE base_sig_version**  
*The base shared image definition's version where the vanilla image exists*  
____

# Contribute

To contribute, please follow the steps from *The DevOps Patterns Wiki* section, [How to contribute](https://dev.azure.com/ATTDevOps/ATT%20Cloud/_wiki/wikis/DevOps%20Automation/436/How-To-Contribute).
