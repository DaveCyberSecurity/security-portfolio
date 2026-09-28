<h1 align="center">The Stolen Identity:</h1> 
<h2 align="center">An OAuth consent-phishing App registration kill chain investigation</h2>

---


## Scenario:

Sometime in the last 24 hours, someone got access into the Mad Hat Labs tenant. Oh naurrrrr....They didn't kick down any doors using any zero-day exploits on anything. They walked right in through the identity plane, and man were they were quiet about it. No alarms were tripped. The Mad Hat Labs security team were NOT notified of any MALFEASANCE. The logs just show a series of perfectly normal sign-ins.

In this exercise, I reconstructed a five-stage OAuth consent-phishing kill chain in a live Azure tenant through forensic analysis of two linked app registrations.

In an OAuth consent phishing scenario, the attacker registers a malicious OAuth application on their infrastructure (in Microsoft Entra ID, Google workspace or some other platform) and sends the user/victim a phishing email. The email contains a link to the seemingly legitimate application and asks the user to authorize the app. If the user clicks on the malicious link, instead of being taken to a suspicious looking domain, they are taken to a legitimate Microsoft or google sign-in page.  The user is presented with a consent screen that requests access. If the user clicks Accept, an access token is created and passed on to the attacker’s infrastructure. This token can then be used to access the tenant with the same level of access that the phished user possesses. The attacker has gained ongoing API-level access without the user noticing and can make API calls on the user's behalf. This kind of attack does not steal credentials or use fake sign-in pages. It never has to defeat MFA because it has piggy-backed a user's access though a trusted identity provider like Microsoft and abused the platform's consent flow.

Once the attacker is in, they will seek to use the access they have gained to issue API calls to find what resources they can exploit. The name of the game at this point is to establish persistence. In the following scenario, the attacker got through by OAuth consent-phishing. Our job is to investigate what happened next.

---

## Environment:
Platform: Live multi-user Azure training tenant

Services and Tools: Azure Portal, Azure Resource manager

Access level: Reader access

---

## Investigation:

### Objective 1: ENTRY
The attacker gained entry by the method of OAuth consent-phishing detailed above. The user completed MFA, clicked Accept to the consent screen and their access token was captured.
The attacker then began to look for resources to exploit and establish persistence. The attacker found what they were looking for in a Legacy Enterprise connection application which the user had ownership access to. The incident team responded and identified the phished user as Carl from accounting. Carl had been left as an owner of the Legacy application by mistake and had never been removed. 


First , we look at the Legacy app under App Registrations. In the Branding and Properties blade, there are internal context notes establishing that Carl was the owner on the Legacy Application. 

![image alt](https://github.com/DaveCyberSecurity/security-portfolio/blob/main/Azure/chapter02-the-stolen-identity/images/1-LegacyApp.png)


## 
### Objective 2: ESCALATE

Since Carl was the owner of the App, the attacker could use Carl's token permissions to make changes to the app to establish a backdoor.

The Certificates and Secrets blade on the Legacy app shows that the attacker created a new Client secret. As can be seen from the following screenshot, a client secret can also be referred to as an application password. The client secret can be used to log in as the application and request a new access token. In doing so, the attacker no longer needs Carl's token to get in. The attacker can use the Client ID, Tenant ID and the new Client Secret to authenticate as the application. 

To further enable persistent access, the expiry date of the Client Secret is set to 12/31/2099. Plenty of time to be up to lots of naughtiness.

![image alt](https://github.com/DaveCyberSecurity/security-portfolio/blob/main/Azure/chapter02-the-stolen-identity/images/2-Secrets-2.png)

##
### Objective 3: PIVOT

By using the new Client secret, the attacker can now perform API calls using the permissions of the application. 

Looking at the API permissions blade of the Legacy application, we see that it has the Directory.Read.All and the User.Read.All permissions. Entra ID user accounts, including administrator accounts, DO NOT have the Directory.Read.All or the User.Read.All permissions. These are Microsoft Graph permissions that are granted to applications/service principles. The user will only have them indirectly through applications using delegated permissions. Carl was a normal user, so gaining access to the Legacy app was a definite escalation of permissions for the attacker and took Carl completely out of the loop.

One thing to note: 

When a standard user is allowed to register new applications in Entra, the newly created app starts with no powerful Microsoft Graph permissions. Creating an app registration does not automatically grant access to tenant data. The **Directory.Read.All** or the **User.Read.All** permissions have to be explicitly requested and consented to administratively.

As you can see from the circled portion 'Grant Admin Consent for Mad Hat Labs' the legacy app was granted those permissions at some point in the past.
#

![image alt](https://github.com/DaveCyberSecurity/security-portfolio/blob/main/Azure/chapter02-the-stolen-identity/images/3-API-Permissions.png)

#

What if that Client secret from Objective 2 on the Legacy App were to be deleted? The attacker's access would go away. 

Looking at the Owner blade for the Legacy App, a new rogue entry has appeared. It is a new Service Principle of a new App registration created by the attacker.
Carl's permissions allow him to register new applications and add owners to the Legacy app. The attacker used Carl's permissions to do just that.
They created a new Service principle and made it an owner of the the Legacy app. 

Now, if the secret is rotated, the attacker can re-credential by navigating to App Registrations -> Legacy App -> Certificates and secrets, add a new client secret, record the secret value.
The attacker can then use the new credential to obtain another OAuth token as the legacy application. This owner app acts as a shadow admin.
#

![image alt](https://github.com/DaveCyberSecurity/security-portfolio/blob/main/Azure/chapter02-the-stolen-identity/images/3-OwnerList.png)



##
### Objective 4: PERSIST

Looking at the Expose an API blade of the Legacy application a custom API scope has appeared.
The attacker used Carl's access to create the scope on the legacy app.
A custom scope tells Entra ID that the Legacy application can act as a secured back end resource that other applications can call.

The attacker wants multiple independent paths of regaining access to the legacy app.
Being able to create a new secret repeatedly from Objective 3 is one path. 
The secret method allows the attacker to make API calls with the Legacy App's permissions.

Publishing a custom API scope to the API blade is another.
The custom API scope method allows the attacker to call upon the Legacy App itself to make the API calls.
The goal is to create a second access path that will survive credential cleanup.

The general format for an API custom scope looks something like this:
api://legacy-app/access_as_user

The attacker's app is then pointed at that link.

#
![image alt](https://github.com/DaveCyberSecurity/security-portfolio/blob/main/Azure/chapter02-the-stolen-identity/images/4-ExposeAnAPI.png)



##
### Objective 5: LOOT
From the attacker's perspective, it gets even better. 

On the Authentication blade of the attacker's new service principle, two new redirect URIs appeared.
The top URI points to the attacker's infrastructure.
The other is a combination of the rogue app client id, the redirect URI and the Expose the API string from Objective 4.
This can be used to launch an entirely new OAuth consent phishing campaign against the tenant users.


![image alt](https://github.com/DaveCyberSecurity/security-portfolio/blob/main/Azure/chapter02-the-stolen-identity/images/5-RedirectURI.png)

---

## Attack Chain Summary
| Stage | Objective | Action |
|---------|---------|---------|
| 1 | Entry | OAuth consent phishing captured Carl's access token |
| 2 | Escalate | New client secret added to Legacy Application |
| 3 | Pivot | Rogue service principal added as owner |
| 4 | Persist | Custom API scope published |
| 5 | Loot | Redirect URIs configured for future phishing |

---

## What surprised me:
I was genuinely surprised (and somewhat horrified at first) that Microsoft Entra ID allows all member users to register applications by default. This behavior dates back to Azure AD's original goal of enabling self-service development and SaaS integration without requiring administrators for every application registration.

When a standard user registers an app, the user:
- Becomes an owner of the application they create.
- Can manage that application's configuration.
- Cannot automatically grant admin-level permissions.
- Still requires administrator approval for permissions that require admin consent.

The good news:
As mentioned in Objective 3, when a standard user is allowed to register new applications in Entra, the newly created app starts with no powerful Microsoft Graph permissions. Creating an app registration does not automatically grant access to tenant data. The **Directory.Read.All** or the **User.Read.All** permissions have to be explicitly requested and consented to administratively. This can be caught by monitoring and alerting when an Administrative consent event takes place.


---

## Findings and recommendations:

The following remediation actions are recommended based on evidence of OAuth consent phishing, application ownership abuse, persistence through application credentials, and malicious app-to-app trust relationships.

---

### 1. Revoke Unauthorized Application Credentials

#### Finding
The attacker established persistent access by creating a client secret on the compromised Legacy Application.

#### Risk
Application credentials allow direct authentication as the application and can remain valid even after user sessions are terminated or passwords are changed.

#### Recommendation
Immediately revoke and remove all unauthorized client secrets and certificates associated with the affected application. Following removal:

- Rotate all remaining legitimate credentials.
- Validate that no additional credentials have been created.
- Verify that no unauthorized certificates remain present.

**Priority:** Critical  
**Owner:** Identity and Access Management (IAM)

---

### 2. Remove Rogue Service Principals from Application Ownership

#### Finding
A malicious service principal was added to the application's Owners list, providing a mechanism to re-establish credentials after remediation.

#### Risk
Application ownership grants administrative control over the application, including the ability to:

- Create new secrets
- Modify permissions
- Alter authentication settings
- Add additional owners

#### Recommendation
Remove all unauthorized service principals and user accounts from the application's Owners list. Validate ownership assignments against approved application administrators and business owners.

**Priority:** Critical  
**Owner:** IAM / Application Governance

---

### 3. Remove Malicious Custom API Scopes

#### Finding
A custom OAuth scope was published through the application's **Expose an API** configuration.

#### Risk
Unauthorized scopes may:

- Provide alternative access paths
- Enable application impersonation
- Facilitate future consent-phishing campaigns
- Create undocumented trust relationships

#### Recommendation
Delete all unauthorized custom API scopes and review the application's API exposure settings to ensure they align with documented business requirements.

**Priority:** High  
**Owner:** IAM / Application Owners

---

### 4. Revoke Existing OAuth Consent Grants

#### Finding
OAuth consent grants may remain active even after application credentials, owners, or scopes have been removed.

#### Risk
Valid consent grants can continue to provide access tokens to previously authorized applications.

#### Recommendation
Explicitly revoke all associated **OAuth2PermissionGrant** objects and verify that both user-level and tenant-wide consent grants have been removed.

Validate that:

- No delegated permissions remain active.
- No active consent relationships exist.
- Access tokens can no longer be issued through the previously authorized application.

**Important:** Containment actions such as deleting secrets, owners, or scopes do **not** automatically remove existing OAuth consent grants.

**Priority:** Critical  
**Owner:** IAM / Security Operations Center (SOC)

---

### 5. Remove Unauthorized Redirect URIs

#### Finding
The attacker configured redirect URIs pointing to external infrastructure.

#### Risk
Malicious redirect URIs can be used to capture authorization codes and OAuth tokens during future authentication flows.

#### Recommendation
Remove all unauthorized redirect URIs and verify that all remaining endpoints are:

- Approved
- Documented
- Business-justified
- Owned and managed by the organization

**Priority:** High  
**Owner:** Application Owners

---

### 6. Review and Reduce Microsoft Graph Permissions

#### Finding
The compromised application possessed powerful Microsoft Graph permissions, including directory-wide read access.

#### Risk
Excessive application permissions increase the impact of application compromise and broaden attacker visibility throughout the tenant.

#### Recommendation
Perform a least-privilege review of all Microsoft Graph permissions assigned to the application.

Actions should include:

- Removing unnecessary permissions.
- Reviewing all admin-consented permissions.
- Requiring periodic recertification of high-risk permissions.
- Documenting business justification for elevated permissions.

**Priority:** High  
**Owner:** IAM / Governance Team

---

### 7. Restrict User Application Registration

#### Finding
The attacker leveraged the ability of a standard user to register a new application and service principal.

#### Risk
Unrestricted application registration allows attackers to create persistence mechanisms and establish unauthorized trust relationships following account compromise.

#### Recommendation
Disable default user application registration where business requirements permit.

Where application registration is required:

- Restrict registration rights to approved groups.
- Implement approval workflows.
- Monitor new app registrations.
- Require periodic review of newly created applications.

**Priority:** High  
**Owner:** IAM

---

### 8. Conduct a Tenant-Wide Application Ownership Review

#### Finding
Excessive or stale ownership assignments enabled the attacker's escalation path.

#### Risk
Application owners possess significant administrative authority that is frequently overlooked during identity governance reviews.

#### Recommendation
Audit all application registrations and enterprise applications to identify:

- Inappropriate owners
- Excessive ownership assignments
- Orphaned applications
- Inactive owners
- Service-principal ownership relationships

Application ownership should be reviewed with the same rigor and frequency as privileged directory role assignments.

**Priority:** High  
**Owner:** IAM / Governance Team

---

### 9. Implement Continuous Monitoring and Alerting

#### Finding
The attacker established persistence through configuration changes that often generate little operational visibility.

#### Risk
Malicious application modifications may remain undiscovered without dedicated monitoring controls.

#### Recommendation
Implement security monitoring and alerting for the following events:

- New client secret creation
- New certificate creation
- Application owner changes
- Service principal owner assignments
- Redirect URI additions or modifications
- Custom API scope creation
- OAuth consent grants
- Administrative consent events
- Newly registered applications

Alerts should be integrated into SOC monitoring workflows and investigated promptly.

**Priority:** High  
**Owner:** SOC / Detection Engineering

---

## What I learned:
At the outset of the investigation, I made the erroneous assumption that Phishing-Resistant MFA could have prevented the OAuth consent phishing scenario. 
Phishing-resistant MFA significantly reduces the risk of credential theft and adversary-in-the-middle (AiTM) phishing attacks. 
However, **it does not, by itself, prevent OAuth consent phishing**.

The reason is simple:
 
- **Phishing-resistant MFA (such as FIDO2 security keys, Windows Hello for Business, or passkeys) protects authentication.**
- **OAuth consent phishing abuses authorization.**


In a consent phishing attack, the attacker does not need the user's password, session cookie, or MFA code. Instead, they convince the user to authorize a malicious application that requests access to organizational resources.










