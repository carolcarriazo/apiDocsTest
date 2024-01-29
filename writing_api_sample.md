## Restarting a mongod Process Using the Ops Manager API

## Overview 

This article guides you through restarting a specific `mongod` process using the Ops Manager API. The method involves updating the **Automation Config** resource in the `processes` array by adding a `lastRestart` parameter with the current timestamp for the cluster's processes. 

Setting this parameter to a specific timestamp prompts the Ops Manager to force a restart of the process at the designated time once you upload the configuration. If you apply this parameter to multiple processes in the same cluster, Ops Manager will sequentially restart the selected processes in the replica set or shards.

**Important**: Always fully test these steps in a non-production environment before applying them to a production cluster.

The basic steps include:

1. Generating API keys for your Ops Manager deployment.
2. Retrieving the automation configuration for the project.
3. Modifying the automation configuration by adding the `lastRestart` parameter to the processes array.
4. Confirming the status of the mongod process through the **Ops Manager Activity Feed** or by executing `ps -ef |grep mongo` on the host.
5. Noting that the `lastRestart` parameter will no longer be visible after the completion of the restart.

## Detailed Steps

Remember, the API key, Ops Manager URL, and other details in this example are from a test environment. Substitute these values with the correct ones from your Ops Manager deployment.

1. **Generate an API Key:** Navigate to **Project** > **Access Manager** -> **Project Access** -> **API Keys** > **Create API Keys** in your Ops Manager project. Remember to store the public / private key pair safely.

2. **Retrieve Automation Configuration:** Use the Get the Automation Configuration endpoint to obtain the project's Automation configuration and save it as a JSON file, typically named `currentAutomationConfig.json`.

   Example request:

   ```
   curl --user "{publicApiKey}:{privateApiKey}" --digest \
   --header "Accept: application/json" \
   --request GET \
   "https://<OpsManagerHost>:<Port>/api/public/v1.0/groups/"
   ```

    Example response:

    ```
    % Total  % Received % Xferd Average Speed Time Time  Time Current Dload Upload Total Spent Left Speed 
    100 106 100 106  0  0 33576 0 35333 
    100 1393k 0 1393k 0 0 29.1M 0 29.1M 
    ```

3. **Modify the Automation Configuration:** Open the `currentAutomationConfig.json` and locate the processes array. Insert the `lastRestart` parameter with the date and time in ISO 8601 format ("YYYY-MM-DDTHH:MM:SS+or-GMT") in UTC, following the `featureCompatibilityVersion` line for the deployment member you wish to restart. Save the file after modifications.

   For example, to schedule a restart at 12:45 PM GMT on Jul 13, 2023:

   ```
   "lastRestart": "2023-07-13T17:45:00+05:30",
   ```

4. **Upload the Updated Configuration:** Use the `HTTP PUT` method with the **Update the Automation Configuration** endpoint to apply the changes in the `currentAutomationConfig_updated.json`.

   Example request:

   ```
   curl --user "{PUBLIC-KEY}:{PRIVATE-KEY}" --digest \
   --header "Accept: application/json" \
   --header "Content-Type: application/json" \
   --include --request PUT \
   "https://<OpsManagerHost>:<Port>/api/public/v1" \
   --data-binary "@/Users/admin/currentAutomationConfig_updated.json"
   ```
    
    Example response:

    ```
    HTTP/1.1 401 Unauthorized 
    WWW-Authenticate: Digest realm="MMS Public API", domain="", 
    Content-Length: 0 
    HTTP/1.1 100 Continue 
    nonce="JuwM4w52 
    HTTP/1.1 200 OK 
    Date: Tue, 05 Sep 2023 06:35:03 GMT 
    Strict-Transport-Security: max-age=0; includeSubdomains; 
    Referrer-Policy: strict-origin-when-cross-origin 
    X-Permitted-Cross-Domain-Policies: none 
    X-Content-Type-Options: nosniff 
    X-MongoDB-Service-Version: gitHash=e6efff793e3f0ff2fb7daa51d70306934dced894; 
    Content-Type: application/json 
    X-Frame-Options: DENY 
    Content-Length: 2
    ```

5. **Verify the Restart:** Monitor the restart process through the **Ops Manager Activity Feed** where you should see the message `Host has restarted`. Alternatively, confirm the `mongod` process's restart timestamp by executing the `ps -ef |grep mongo` command on the MongoDB host.