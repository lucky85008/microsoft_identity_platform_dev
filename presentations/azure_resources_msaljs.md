---
title: 'Authenticating to Azure with MSAL.js'
author: 'Part 3 of 4 in the [Develop JavaScript Applications with the Microsoft Identity Platform](https://github.com/msusdev) series'
---

# Authenticating to Azure with MSAL.js

## Overview

### About us

Co-Presenter Name

☁️ *Co-Presenter Title*

Co-Presenter Name

☁️ *Co-Presenter Title*

> For questions or help with this series: <MSUSDev@Microsoft.com>

All demos and source code available online:

> <https://github.com/MSUSDEV/microsoft_identity_platform_dev>

## Setting the scene

### Series roadmap

* ~~Session 1:~~
  * ~~Authenticating users in JavaScript apps with MSAL.js~~
* ~~Session 2:~~
  * ~~Discover Microsoft Graph Toolkit Components~~
* **Session 3:**
  * **↪️ Authenticating to Azure with MSAL.js**
* Session 4:
  * The Microsoft Graph SDK for JavaScript

### Today's agenda

1. Use msal-node library in a Node server-side app
1. Use default azure credential provider for Azure SDK 
1. Create custom credential provider

## Demo: *Accessing Azure resource using MSAL.js and a Node.js server-side app*

::: notes

##### Create Node server-side app

> Register app with **secret**, **User.Read.All**, and **admin constent**.

```bash
npm init
```

```json
{
  "name": "servernodemsal",
  "main": "app.js",
  "type": "module",
  "scripts": {
    "start": "node app.js"
  }
}
```

##### Configure MSAL.js 2.0 (Node variant)

```bash
npm install @azure/msal-node --save
```

```javascript
import { ConfidentialClientApplication } from '@azure/msal-node';
```

```javascript
const config = {
    auth: {
        clientId: '<client-id>',
        authority: 'https://login.microsoftonline.com/<tenant-id>',
        clientSecret: '<client-secret>'
    }
};

var client = new ConfidentialClientApplication(config);

var request = {
    scopes: [ 'https://graph.microsoft.com/.default' ]
};

let response = await client.acquireTokenByClientCredential(request);

console.dir(response);
```

##### Query Microsoft Graph

```bash
npm install node-fetch --save
```

```javascript
import fetch from 'node-fetch';
```

```javascript
let query = await fetch('https://graph.microsoft.com/v1.0/users', {
    headers: {
        'Authorization': 'Bearer ' + response.accessToken
    }
});
let json = await query.json();
console.dir(json);
```

##### Manipulate Azure Storage

> Add Azure Storage **user_impersonation** permission and grant **admin consent**. Also use RBAC to add AAD app reg as a **Storage Blob Data Contributor**.

```bash
npm install @azure/storage-blob --save
```

```javascript
import { BlobServiceClient }  from '@azure/storage-blob';
```

```bash
npm install @azure/identity --save
```

```javascript
import { DefaultAzureCredential } from '@azure/identity';
```

```javascript
var request = {
    scopes: [ 'https://storage.azure.com/.default' ]
};
```

```javascript
var client = new BlobServiceClient('https://<storage-account>.blob.core.windows.net/', new DefaultAzureCredential());

let container = client.getContainerClient('democontainer');
await container.createIfNotExists();
```

```bash
npm install dotenv --save-dev
```

```javascript
import dotenv from 'dotenv';
```

```env
AZURE_CLIENT_ID ="<client-id>"
AZURE_TENANT_ID="<tenant-id>"
AZURE_CLIENT_SECRET="<client-secret>"
```

```javascript
dotenv.config();
```

##### Create custom token credential

```bash
npm install @azure/core-auth --save
```

```javascript
class MyAzureCredential {
    async getToken(requestedScopes) {
        const config = {
          auth: {
            clientId: '<client-id>',
            authority: 'https://login.microsoftonline.com/<tenant-id>',
            clientSecret: '<client-secret>'
          }
        }
        var client = new ConfidentialClientApplication(config);
        var request = {
            scopes: Array.isArray(requestedScopes) ? requestedScopes : [requestedScopes]
        };        
        let response = await client.acquireTokenByClientCredential(request);
        return {
            token: response.accessToken,
            expiresOnTimestamp: response.expiresOn.getTime()
        }
    };
}
```

```javascript
var client = new BlobServiceClient('https://<storage-account>.blob.core.windows.net/', new MyAzureCredential());

let container = client.getContainerClient('examplecontainer');
await container.createIfNotExists();
```

##### Use custom token credential with

```bash
npm install @azure/cosmos --save
```

```javascript
import { CosmosClient } from '@azure/cosmos';
```

```javascript
var client = new CosmosClient({
   aadCredentials: new MyAzureCredential,
   endpoint:  'https://<account-name>.documents.azure.com:443/'
});

let response = await client.getDatabaseAccount();

console.dir(response);
```

:::
