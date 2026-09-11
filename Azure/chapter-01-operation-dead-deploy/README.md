# Investigating an Azure Governance Failure: Operation Dead Deploy

## Scenario
An intern with temporary Contributor access deployed a "test environment" over a weekend, cut every corner, and left. You come in Monday as the on-call engineer with Reader access and have to reconstruct what happened and why governance did not stop it.

In essence, a resource and associated resource group was discovered in the Azure Tenant that violated several compliance policies. The purpose of this investigation was to find out when the resources were deployed, by whom and how the resources were able to be created in violation of policy. It was also important to determine whether this was the only affected resource in the tenant.

## Environment
Platform: Live multi-user Azure training tenant

Services and Tools: Azure Portal, Azure Resource manager

Access level: Reader access

## Investigation
1. Starting with the resource list in the Azure tenant, I found a storage account that had been created in a resource group with a strange name. The resource group's name did not follow Microsoft's           recommended naming convention for resource groups in the tenant.
   Resource group: redacted

![image alt](https://github.com/DaveCyberSecurity/security-portfolio/blob/main/Azure/chapter-01-operation-dead-deploy/images/resource-group.png)

2. I examined the storage group's associated tags and found the owner tag was set to: 
   intern-redacted

   In addition, an associated intern-flag tag was added by the intern as part of the normal creation process. 
  ![image alt](https://github.com/DaveCyberSecurity/security-portfolio/blob/main/Azure/chapter-01-operation-dead-deploy/images/intern-flags.png)

This indicated that the storage group had been created by an intern account. 

3. Next, I examined the deployment associated with the misnamed Resource group. The deployment name had the word intern in the string.

   This was further indication that its authorship was an intern account.
	The Last modified date on the Deployment indicates when it was created.

4. In an attempt to determine whether or not this was the only affected resource, I examined the list of resource groups. With just a precursory glance at the list, one could tell that the misnamed         resource group was the only one that had been created improperly from a naming perspective. Every other resource group conformed to proper naming convention.

5. Policies are ostensibly in place in the environment to prevent the creation of resource groups that do not conform to proper naming standards. Why was this not caught by policy? I went to the new        resource group and examined the policies associated with it. I found the resource group and its contained resource to be in violation of 1 naming convention policy and also 9 security policies           associated with the Microsoft cloud security benchmark.


6. To determine what the naming policy was actually doing, I examined the JSON definition of the naming convention policy.
   It specified that if the name field did not start with the following   characters: "rg-” then a specified action would take place.
   The choices available were to deny the effect, audit the effect or to disable the policy.
   The naming convention policy effect parameter Default Value was set to audit.
   This allowed the resource to be created incorrectly while being flagged as non-compliant.

7. The next step was to determine whether this was the same situation with the other policy violations for the storage account. I examined each sub-category of Microsoft cloud security benchmark and exported the policies to .csv to make the information more human-readable. A similar Audit effect parameter type was found on each of those policies as well. 

Policy Group Title:
NS-2 Secure cloud services with network controls - 6 policies in violation.
All effect types are set to either Audit or AuditIfNotExists.






## What broke / what surprised me
.

## Findings and recommendations


## What I learned

