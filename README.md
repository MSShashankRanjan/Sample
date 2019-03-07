# Retreiving KeyVault secrets from Azure Function V2 using JavaScript HTTP trigger

<H3>This is sample document where you need to retrieve the secrets from the KeyVault using Azure HTTP Trigger Function V2.</H3>

As we see in Azure Function V1, we don't use "async" functionality and in V2 function it genereally uses "async".
However with the JavaScript code we tend to use "callback" and "async" together. Which in turn results in misbehave the function where you see values doesn't gets logged properly.

<H2>Note:- Declaration of callbacks and the async function should be avoided. </H2>

To have your execute as expected, we recommend that you change the code to use all callbacks or all async & await / Promises and not mix the two (this is a JavaScript quirk, with the added confusion of “async” being promise-specific being introduced in Node.js v8+).
 
To change the code to use all callbacks, you would need to remove the “async” function declaration and use the context.done callback. Your code should be the exact same code as it was in V1.
 
To change the code to use all async & await / Promises, you would need to use Promise-returning API’s from ‘azure-keyvault’.

Script - "index.js"

<H1>*********************************************************************************************************************</H1>

```
const KeyVault = require('azure-keyvault');
const AuthenticationContext = require('adal-node').AuthenticationContext;

module.exports = function (context, req) {
    const clientId = "<your client id or the application id";
    const clientSecret = "<application secret>";
    // Getting authorizationValue
    
    const authenticator = function (challenge, callback) {

        // Create a new authentication context.
        const authcontext = new AuthenticationContext(challenge.authorization);

        // Use the context to acquire an authentication token. 
        authcontext.acquireTokenWithClientCredentials(challenge.resource, clientId, clientSecret, function (err, tokenResponse) {
            if (err) {
                context.done(err); // see docs here: https://docs.microsoft.com/en-us/azure/azure-functions/functions-reference-node#contextdone-method
            }

            // Calculate the value to be set in the request's Authorization header and resume the call.
            const authorizationValue = tokenResponse.tokenType + ' ' + tokenResponse.accessToken;
            callback(null, authorizationValue);
        });
    }

    const credentials = new KeyVault.KeyVaultCredentials(authenticator, null);
    const client1 = new KeyVault.KeyVaultClient(credentials);
    const keyvaulturl = "<KeyVault URL>";
    const secretName = "<secret name from KeyVault>";
    const secretVersion = "<Secret Version>";

    client1.getSecret(keyvaulturl, secretName, secretVersion, function (err, result) {
        if (err) {
            context.done(err); // see docs here: https://docs.microsoft.com/en-us/azure/azure-functions/functions-reference-node#contextdone-method
        }

        const rs = result.value.toString();
        context.log(result.value);

        context.res = {
            // status: 200, /* Defaults to 200 */
            body: "Hello " + rs
        };

        context.log('JavaScript HTTP trigger function processed a request 1.');
        context.log("My Value:" + rs);

        context.done(); // see docs here: https://docs.microsoft.com/en-us/azure/azure-functions/functions-reference-node#contextdone-method
    });
};
```
