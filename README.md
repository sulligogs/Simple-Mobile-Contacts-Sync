# Simple-Mobile-Contacts-Sync
v1.0

## Purpose
Synchronises company users as contacts to their mobile phones.  Could be useful for small businesses with a limited infrastructure.  Saves the workforce from having to do it manually.

## Motive
I started getting interested in the Microsoft Graph API and used this as a solution to teaching myself how it works.  Also gave me a chance to stretch my PowerShell legs.  It uses the Contacts.ReadWrite and User.Read.All application permissions.

I know there are more elegant methods of achieving this solution with Dynamics 365 and/or Intune.

## Mechanics
Each enabled Azure AD user is pushed as a personal contact to each other's Exchange Online mailbox.  Where they have the Outlook mobile app installed with its contact sync enabled, they then get pushed further to their mobile phone's contact app.

### Usage
powershell -File *path*\SMCS.ps1 -ClientID client id -TenantID tenant id -Thumbprint certificate id [-GroupFilter group id] [-CategoryRemoval category]

### Parameters
**-ClientID client id** is the application ID for this app.  When it's registered with an organisation's Azure AD this will be provided to you.<br>
**-TenantID tenant id** is the unique tenant ID that an organisation's Azure environment receives when it is setup.<br>
**-Thumbprint certificate id** is the public key that is provided when building an app certificate.<br>
**-GroupFilter group id** is a Unified Group whose members will have their contacts sync'd to each other in that group.  Useful if you don't want contact syncing for every employee, but also as a test group before deploying live.  The value is the Object Id of the group from Azure AD.  If unspecified then all users in the organisation will be sync'd.<br>
**-CategoryRemoval category** may be useful if you are migrating away from an existing contacts sync solution.  Any contacts that match the catergory specified will be removed from each users' personal contacts.<br>

#### Prerequisites

A Windows computer with PowerShell 5.1 that will perpetually run the script.<br>
Each user must have an O365 Exchange Online mailbox.<br>
Each phone must have the Outlook mobile app installed.

#### How To
To create an app certificate please look at https://gist.github.com/nicolonsky/e3f94acd49d51ab66ca3a4c9a7ce37a8 <br>
To apply the certificate to your Azure AD environment please look at https://laurakokkarinen.com/authenticating-to-office-365-apis-with-a-certificate-step-by-step/<br>
To get a group's Object Id from the Azure Portal look at step 3 of https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/active-directory-groups-create-azure-portal
