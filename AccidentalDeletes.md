# Preview scope
The preview allows you to prevent unexpected deletions by defining a threshold for deletions. When the number of deletions excceds the defined threshold, the provisioning job will go into quarantine and trigger an email. You can choose to accept the deletions or reject them. 

# Known Limitations
* Changing scope from sync all users and groups to assigned users and groups should trigger disables for users and groups that aren't assigned. Those disables aren't covered by the current preview

# Instructions
1) Configure your application for provisioning as you would today
2) Sign into graph explorer as a global administrator
3) Perform a get secrets operation

```HTTP
GET https://graph.microsoft.com/beta/servicePrincipals/{id}/synchronization/secrets

{"value":[{"key":"SyncNotificationSettings","value":"{\"Enabled\":true,\"Recipients\":\"johndoe@xyz.com\",\"DeleteThresholdEnabled\":true,\"DeleteThresholdValue\":50}"}]}

```


4) Perform a PUT secrets operation

```HTTP
PUT https://graph.microsoft.com/beta/servicePrincipals/{id}/synchronization/secrets

{"value":[{"key":"SyncNotificationSettings","value":"{\"Enabled\":true,\"Recipients\":\"johndoe@xyz.com\",\"DeleteThresholdEnabled\":true,\"DeleteThresholdValue\":50}"}]}

```

5) Trigger disables / deletes. There are a umber of ways of doing this
  - Disabling a user in Azure AD through the Azure Portal / Powershell / MS Graph
  - Deleting a previously disabled user in Azure AD through the Azure Portal / Powershell / MS Graph
  - Changing a scoping filter so the user or group goes out of scope
  - Unassigning the user from the app

6) Allow the disables / deletes to be processed
```HTTP
POST https://graph.microsoft.com/beta/servicePrincipals/{id}/synchronization/jobs/{jobId}/restart
Authorization: Bearer <token>
Content-type: application/json

{
   "criteria": {
       "resetScope": "ForceDeletes"
   }
}
```

#
