# Goal
We want a server to get the files uploaded to a shared onedrive folder.

# Problems
We cannot use any interactive authentication method, as the server is supposed to run in an automated fashion, and in different time instants which requires to login every time the process is started.

# Solutions

## Application Permissions
This solution gives too many permissions to our application, as it would be relative to a whole tenant, and we only want permissions for a single shared folder that belongs to a drive.

## Delegated Permissions
Due to Maersk's policies we cannot authenticate a user without 2FA. 
This solution involves creating a new user, entity which is to be associated to this application, belonging to the E-Fulfilment Team Group. 
We would also need permissions to authenticate this user without 2FA, using the username and password method. 
Additionally, we would require Admin Consent for the delegated User.Read and Files.ReadWrite API delegated permissions.

We look forward to hearing from you,
Data Engineering Team