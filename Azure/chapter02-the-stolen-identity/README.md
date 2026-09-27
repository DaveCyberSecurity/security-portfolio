# The Stolen Identity: #
### An OAuth consent-phishing App registration kill chain investigation ###



## Scenario
Sometime in the last 24 hours, someone got access into the Mad Hat Labs tenant. Oh naurrrrr....They didn't kick down any doors using any zero-day exploits on anything. They walked right in through the identity plane, and man were they were quiet about it. No alarms were tripped. The Mad Hat Labs security team were NOT notified of any MALFEASANCE. The logs just show a series of perfectly normal sign-ins.

In this exercise, I reconstructed a five-stage OAuth consent-phishing kill chain in a live Azure tenant through forensic analysis of two linked app registrations.

In an OAuth consent phishing scenario, the attacker registers a malicious OAuth application on their infrastructure (in Microsoft Entra ID, Google workspace or some other platform) and sends the user/victim a phishing email. The email contains a link to the seemingly legitimate application and asks the user to authorize the app. If the user clicks on the malicious link, instead of being taken to a suspicious looking domain, they are taken to a legitimate Microsoft or google sign-in page.  The user is presented with a consent screen that requests access. If the user clicks Accept, an access token is created and passed on to the attacker’s infrastructure. This token can then be used to access the tenant with the same level of access that the phished user possesses. The attacker has gained ongoing API-level access without the user noticing and can make API calls on the user's behalf. This kind of attack does not steal credentials or use fake sign-in pages. It never has to defeat MFA because it has piggy-backed a user's access though a trusted identity provider like Microsoft or Google's authorization flow.

Once the attacker is in, they will seek to use the access they have gained to issue API calls to find what resources they can exploit. The name of the game at this point is to establish persistence. In the following scenario, the attacker got through by OAuth consent-phishing. Our job is to recreate what happened next.


## Environment
Platform: Live multi-user Azure training tenant

Services and Tools: Azure Portal, Azure Resource manager

Access level: Reader access


## Investigation

## Objective 1: ENTRY
The attacker gained entry by the method of OAuth consent-phishing detailed above. The user completed MFA, clicked Accept to the consent screen and their access token was captured.
The attacker then began to look for resources to exploit and establish persistence. The attacker found what they were looking for in a Legacy Enterprise connection application which the user had ownership access to. The incident team responded and identified the phished user as Carl from accounting. Carl had been left as an owner of the Legacy application by mistake and had never been removed. 


First , we look at the Legacy app under App Registrations. In the Branding and Properties blade, there are internal context notes establishing that Carl was the owner on the Legacy Application. 

![image alt](https://github.com/DaveCyberSecurity/security-portfolio/blob/main/Azure/chapter02-the-stolen-identity/images/1-LegacyApp.png)


## 
## Objective 2: ESCALATE

Since Carl was the owner of the App, the attacker could use Carl's token permissions to make changes to the app to establish a backdoor.
The Certificates and Secrets blade on the Legacy app shows that the attacker created a new Client secret. The client secret can be used to log in as the application and request
a new access token. In doing so, the attacker no longer needed Carl's token to get in.

![image alt](https://github.com/DaveCyberSecurity/security-portfolio/blob/main/Azure/chapter02-the-stolen-identity/images/2-Secrets-2.png)

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
