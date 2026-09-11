# Investigating an Azure Governance Failure: Operation Dead Deploy

## Scenario
An intern with temporary Contributor access deployed a "test environment" over a weekend, cut every corner, and left. You come in Monday as the on-call engineer with Reader access and have to reconstruct what happened and why governance did not stop it.

In essence, a resource and associated resource group was discovered in the Azure Tenant that violated several compliance policies. The purpose of this investigation was to find out when the resources were deployed, by whom and how the resources were able to be created in violation of policy. It was also important to determine whether this was the only affected resource in the tenant.

## Environment
Platform: Live multi-user Azure training tenant

Services and Tools: Azure Portal, Azure Resource manager

Access level: Reader access

## Investigation
Starting with the resource list in the Azure tenant, I found a storage account that had been created in a resource group with a strange name. The resource group's name did not follow Microsoft's recommended naming convention for resource groups in the tenant.
Resource group: redacted

![image alt](https://github.com/DaveCyberSecurity/security-portfolio/blob/main/Azure/chapter-01-operation-dead-deploy/images/resource-group.png)


## What broke / what surprised me
.

## Findings and recommendations


## What I learned

