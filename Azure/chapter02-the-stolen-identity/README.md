# The Stolen Identity: #
### An OAuth consent-phishing App registration kill chain investigation ###



## Scenario
Sometime in the last 24 hours, someone got access into the Mad Hat Labs tenant. Oh naurrrrr....They didn't kick down any doors using any zero-day exploits on anything. They walked right in through the identity plane, and man were they were quiet about it. No alarms were tripped. The Mad Hat Labs security team were NOT notified of any MALFEASANCE. The logs just show a series of perfectly normal sign-ins.

In this exercise, we reconstructed a five-stage OAuth consent-phishing kill chain in a live Azure tenant through forensic analysis of two linked app registrations.

In an OAuth consent phishing scenario, the attacker registers a malicious OAuth application on their infrastructure and sends the user/victim a phishing email. The email contains a link to the seemingly legitimate application and asks the user to authorize the app. If the user clicks on the malicious link, instead of being taken to a suspicious looking domain, they are taken to a legitimate Microsoft or google sign-in page.  The user is presented with a consent screen that requests access. If the user clicks Accept, an access token is created and passed on to the attacker’s infrastructure. This token can then be used to access the tenant with the same level of access that the phished user possesses. 


## Environment
Platform: Live multi-user Azure training tenant

Services and Tools: Azure Portal, Azure Resource manager

Access level: Reader access


## Investigation

## Objective 1: ENTRY



![image alt](https://github.com/DaveCyberSecurity/security-portfolio/blob/main/Azure/chapter02-the-stolen-identity/images/1-LegacyApp.png)

## 
## Objective 2: ESCALATE

![image alt](https://github.com/DaveCyberSecurity/security-portfolio/blob/main/Azure/chapter02-the-stolen-identity/images/2-Secrets.png)

##
## Objective 3: PIVOT


![image alt](https://github.com/DaveCyberSecurity/security-portfolio/blob/main/Azure/chapter02-the-stolen-identity/images/3-API-Permissions.png)


![image alt](https://github.com/DaveCyberSecurity/security-portfolio/blob/main/Azure/chapter02-the-stolen-identity/images/3-OwnerList.png)



##
## Objective 4. PERSIST

![image alt](https://github.com/DaveCyberSecurity/security-portfolio/blob/main/Azure/chapter02-the-stolen-identity/images/4-ExposeAnAPI.png)



##
## Objective 5. LOOT

![image alt](https://github.com/DaveCyberSecurity/security-portfolio/blob/main/Azure/chapter02-the-stolen-identity/images/5-RedirectURI.png)
