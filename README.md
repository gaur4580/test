# Layered Pipeline

# Overview 
Stratum is a mono-repo that contains layers, kits and automation scripts of layers. A layer is a logical grouping of resources that take into consideration:

- Layer can be as simple as single resource type and it can also be grouping of different layers to compose your own custom layer
- Segregation of resource provisioning per layer allows RBAC per different units aka separation of responsibilities
- Service Contract aka output/input between layers initialize variables during deployment
- Layers encourages separation of state file based on resource life cycle. That adds flexibility during troubleshooting, now you can focus on one layer at a time.
Pipeline is flexible enouph to allow rerun specific layer as many times as needed to be.
- Variables are scoped per layers in $env-variables.tf file

# Prerequisite

- ADO agent needs to be deployed using [att-onboarding-devops-services
](https://dev.azure.com/ATTDevOps/ATT%20Cloud/_git/att-onboarding-devops-services
)
- Provide Contributor role to *ado VMSS* to storage account where TF state file need to be saved 

- Verify terraform and unzip binaries installed in ado agent VMSS
- Create service Connection for ado agent to connect to Azure portal using [Service Principal Doc
](https://dev.azure.com/ATTDevOps/ATT%20Cloud/_wiki/wikis/Optimization%20Wiki/1106/ADO-Service-Principal-Creation-and-Permissions-Guide
)
- Create ADO agent pool [Click here](https://dev.azure.com/ATTDevOps/ATT%20Cloud/_wiki/wikis/ATT-Cloud.wiki/670/Create-Azure-Self-Hosted-Build-Agent-(Deprecated))

# How to deploy new Enviornmet!

  - Create a new branch from master
  - Change **environment: dev** in */kits/jumpstart-vm/Pipelines/Pipeline.<env>.yaml* to **environment: <env>** **if required**
  - Change */kits/jumpstart-vm/Pipelines/Varibale.<env>.yaml* and */kits/jumpstart-vm/Pipelines/Variable.yaml* **if required**
  - Change the **name** field in all the *var-<layer>.auto.tfvars* files present here */kits/jumpstart-vm/Layers/<env>/var-<layer>.auto.tfvars*
  - If you want to deploy VM make sure your **vm-name** should be same as local variable name decalared in *var-virtualmachine.tf*
   ```
**/kits/jumpstart-vm/Layers/<env>/var-virtualmachine.tf**
locals {
  linux_image_ids = {
    "ABCD-VM1"  = "XXXXXXX(Resource ID of shared image version"
    "ABCD-VM2"  = "XXXXXXX(Resource ID of shared image version"
  }
}
```
```
**/kits/jumpstart-vm/Layers/<env>/var-virtualmachine.auto.tfvars**
linux_vms = {
  vm1 = {
    name                             = "ABCD-VM1"
    vm_size                          = "Standard_DS1_v2"
    assign_identity                  = true
    avaialability_set_key            = null
    .
    .
    .
    },
vm2 = {
    name                             = "ABCD-VM2"
    vm_size                          = "Standard_DS1_v2"
    assign_identity                  = true
    avaialability_set_key            = null
    .
    .
    .
    }
}
```

# Note:-
- If you want to create PE of any resource make sure it has tag **pe_enable = true**
- **[error]No agents were found in pool <agent pool name> Configure an agent for the pool and try again** that means agent is not registered in agent pool.Create ADO agent pool [Click here](https://dev.azure.com/ATTDevOps/ATT%20Cloud/_wiki/wikis/ATT-Cloud.wiki/670/Create-Azure-Self-Hosted-Build-Agent-(Deprecated))
- **[error]error loading the remote state: Error retrieving keys for Storage Account....** . ADO agent VM is not able to access storage account to download the tfstate file.Provide Contributor access to ado agent VMSS to storage account where you are storing your TFstate file
- If your pipeline is suceesfull but your resources are not deployed make sure **skip: false** under *stages* section is uncommented in Pipeline.<env>.yaml