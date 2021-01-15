# Preview scope
The preview allows you to prevent accidental deletions. You can define a threshold, that if exceeded, would cause the provioning job to stop executing and prompt for an admin to confirm that the deletions are expected. 

# Testing the preview capability
## Test environment
It is okay to use this in a production tenant, but you should create a second enterprise application for testing and point that to a **non production** instance of your application. 

## Identifiers required for testing
* The servicePrincipal id can be found by navigating to your enterprise application and navigating to the properties page
* The jobID can be found in the provisioning configuration page in the progress bar under "view technical information" 

## Test steps
1) Configure your application for provisioning as you would today
2) Provision users into your application
3) Sign into graph explorer as a global administrator
4) Perform a get secrets operation

```HTTP
GET https://graph.microsoft.com/beta/servicePrincipals/{id}/synchronization/secrets

```


5) Copy the response from the previous request and add the 'DeleteThresholdEnabled' and 'DeleteThresholdValue' properties. Ensure that the 'DeleteThresholdEnabled' property is set to true and specify a threshold value. 

```HTTP
PUT https://graph.microsoft.com/beta/servicePrincipals/{id}/synchronization/secrets

{"value":[{"key":"SyncNotificationSettings","value":"{\"Enabled\":true,\"Recipients\":\"johndoe@xyz.com\",\"DeleteThresholdEnabled\":true,\"DeleteThresholdValue\":50}"}]}

```

6) Trigger disables / deletes. There are a umber of ways of doing this
    * Disabling a user in Azure AD through the Azure Portal / Powershell / MS Graph
    * Deleting a previously disabled user in Azure AD through the Azure Portal / Powershell / MS Graph
    * Changing a scoping filter so the user or group goes out of scope
    * Unassigning the user from the app

7) Allow the disables / deletes to be processed
```HTTP
POST https://graph.microsoft.com/beta/servicePrincipals/{id}/synchronization/jobs/{jobId}/restart

{
   "criteria": 
   {
       "resetScope": "ForceDeletes"
   }
}
```

# Known Limitations
* Changing scope from sync all users and groups to assigned users and groups should trigger disables for users and groups that aren't assigned. Those disables aren't covered by the current preview
