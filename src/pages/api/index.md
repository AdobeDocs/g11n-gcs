# Globalization Content Service APIs

What do you want to understand today?

1. [Overview of Globalization Content Service APIs](#overview)
2. [APIs for Globalization Partners](#apis-for-globalization-partners)
3. [Get All Assets API](#get-all-assets-api)
4. [Download Individual Asset](#download-individual-asset)
5. [Manual Translation of Downloaded Asset](#manual-translation-of-downloaded-asset)
6. [Upload Translated xliff to GCS Azure Storage](#upload-translated-xliff-to-gcs-azure-storage)
7. [Initiate Asset Locale Completion in GCS](#initiate-asset-locale-completion-in-gcs)
8. [Cancel Individual Asset API](#cancel-individual-asset-api)

## Overview

Globalization Content Service APIs are divided into four categories: admin, asset, project, and task.

### Admin-related APIs

- Onboard a new enterprise to Globalization Content Service. Register a new organization or retrieve an existing organization.
- Authenticate a user by getting IMS orgs and groups for the specified userToken.
- Add or retrieve content systems, products, locales, service providers, or workflows.
- Find assets and get scoping reports.

### Asset-related APIs

- Get all assets for specified projects and tasks. Dive deeper and retrieve asset details that include info on locales, projects, tasks, and tenant Id.
- Based on where you are in the localization workflow, you can update an asset, preview an asset, and then update and complete the asset or cancel the asset and restart localization process.

### Project-related APIs

- Create or update a new project.
- Get a list of all projects that you can access or dive deeper and get the project details.
- Plan your work by getting all projects with task counts.

### Task-related APIs

- Create, retrieve, or update a task.
- Want to plan how many tasks you have, easy get a list of all tasks.
- Fine-tune your search by looking for assets or tasks that meet certain criteria, such as locale, project name, status, or workflow. You can also zoom in on a task and retrieve more information.
- Create a new task asynchronously for batch processing or out of sync scheduled localization tasks. Update these tasks asynchronously.

> **Note**: See Globalization Content Service **API Cheat Sheet** for a list of all APIs, including their path, parameters, method, and responses.

# APIs for Globalization Partners

A localization partner typically uses two types of APIs.

### One time setup

Register to Globalization Content Service and setup a task polling framework/connector. See [how to register to Globalization Content Service](../partner/index.md#partner-onboarding-the-complete-workflow).

### APIs for continuous localization

These include:

- [Get all assets for the project](#get-all-assets-for-the-project)
- [Download Individual Asset](#download-individual-asset)
- [Manual Translation of Downloaded Asset](#manual-translation-of-downloaded-asset)
- [Upload Translated xliff to GCS Azure Storage](#upload-translated-xliff-to-gcs-azure-storage)
- [Initiate Asset Locale Completion in GCS](#initiate-asset-locale-completion-in-gcs)
- [Cancel Individual Asset API](#cancel-individual-asset-api)

[](#_Update_Service_Provider_1)In the section below, you will learn about these APIs.

## Get All Assets API

This API finds all the assets for the specified project, task and target locale received in an IO Event. It returns information such as locale name, strings to be localized, last modified information and metadata.

`Path: /v1/projects/{project}/tasks/{task}/assets/{targetLocale}?tenantId={tenantId}`

### Key Parameters

1. **tenantId**: The unique ID of the tenant within a customer org.
2. **projectId**: The unique id for the project under which tasks are created.
3. **taskId**: A unique id for the translation task.
4. **targetLocale**: Locale in which translation has to be done.

### Request URL

```java
curl --request GET \
  --url 'https://gcs.adobe.io/v1/projects/<projectId>/tasks/<taskId>/assets/<targetLocale>?tenantId=<tenant_id>' \
  --header 'Authorization: Bearer eyJhbGciOiJS....' \
  --header 'x-api-key: <client_id>'
```

### Responses

You get the following responses:

1. **200 OK**: The 200 OK status code indicates that the request has been processed successfully on the server
1. **401 Unauthorized**: The 401 unauthorized error status response code indicates that the request was not processed because the destination resource's authentication credentials were invalid. Reauthenticating with right credentials is recommended.
1. **403 Forbidden**: The 403 Forbidden error status code indicates that the server has received the request but has refused to allow it. The client may have insufficient rights to a resource.
1. **404 Not Found**: The 404 error is returned when the server can't find the requested resource. It could be broken or dead link because of file movements or entering a non-existent task name.

### Understanding the successful response

The following is the successful response with the 200 Response code.

```java
{
    "service": "GCS",
    "response": [
        {
            "name": "<assetName>",
            "assetId": "<assetId>",
            "sourceUrl": "https://gcsstorageprod.blob.core.windows.net/gcs/uploadFiles/<tenant_id>/<AssetName>?sig=TsQ2n...eBQ%3D&se=2023-12-07T15%3A49%3A24Z&sv=2019-02-02&sp=r&sr=b",
            "status": "IN_TRANSLATION",
            "mimeType": "application/json",
            "createdDate": "2023-12-07T05:49:55.945Z",
            "assetLocales": [
                {
                    "locale": "ja-JP",
                    "status": "IN_TRANSLATION",
                    "updatedDate": "2023-12-07T05:50:14.459Z"
                }
            ],
            "assetUrls": [
                {
                    "locale": "en-US",
                    "url": "https://gcsstorageprod.blob.core.windows.net/gcs/<tenant_id>/<ProjectId>/<TaskId>/normalized/<AssetName>/en-US/<AssetName>.xlf?sig=zi%2B5Z...EE%3D&se=2023-12-08T21%3A44%3A11Z&sv=2019-02-02&sp=r&sr=b",
                    "createdDate": "2023-12-07T05:50:04.958Z",
                    "urlType": "NORMALIZED"
                }
            ]
        }
    ],
    "success": true
}
```

Here are the key sections in the response:

- **assetId**: A unique ID that is used by the content system to track assets if needed.
- **tenantId**: The unique ID of the tenant within a customer org.
- **taskId**: A unique id for the translation task
- **projectId**: The id of the project containing the tasks
- **sourceURL**: Publicly available URLs (S3 or Azure) for downloading the source asset, can be located in response object with parameter "sourceUrl". This is just for reference and not to be consumed for translation process.
- **status**: 200 ok, indicating request has been executed successfully
- **mimeType**: Defines the nature and format of the asset (media type)
- **createdDate**: Date when the asset was created; represented in ISO 8601 spec; e.g. format 2021-08-05T18:35:58Z
- **assetMetadata**: Details of the asset name and asset path
- **assetLocale**: Information that helps define the source asset language. Details locale as fr-FR, status as Created and updatedDate value.
- **assetUrls/url**: It is important to note here is that the actual asset URL to be used by the partner for translation is the "url" parameter in the response object under "assetUrls", which is in NORMALIZED state. The unique object key to locate this asset is below,

```java
<tenant_id>/<ProjectId>/<TaskId>/normalized/<AssetName>/en-US/<AssetName>.xlf
```

> **Note**: To know more about the Get Asset API, see [Get All Assets in Swagger](https://gcscore-dev-va7.stage.cloud.adobe.io/swagger-ui/index.html#/Asset%20Service/getAssetsUsingGET).

## Download individual asset

This Get API donwloads an existing asset for a particular object key.
Here, Object key referes to a location or unique path(in the GCS storage) for an asset.

### Request URL

```java
curl --request GET \
  --url 'https://gcs.adobe.io/v1/assetContent?tenantId=<tenant_id>&objectKey=<tenant_id>/<ProjectId>/<TaskId>/normalized/<AssetName>/en-US/<AssetName>.xlf' \
  --header 'Authorization: Bearer eyJhbGciOiJSU....' \
  --header 'x-api-key: <client_id>'
```

### Successful response

The following is the successful response which is a byte stream with the 200 Response code.

```java
Content of asset as byte array with Content-Type as    "application/octet-stream;charset=UTF-8"
```

## Manual translation of downloaded asset

Manual translation of xliff asset downloaded in the previous step, offline by the translator.

## Upload translated xliff to GCS Azure Storage

This Post API is used to upload the translated xliff to GCS Azure Storage (Partner can use their own storage and provide signed url which is accessible to GCS).

### Request URL

```java
curl --request POST \
  --url https://gcs.adobe.io/v1/uploadToStorage \
  --header 'Authorization: Bearer eyJhbGci...O_meqYghG0IX5jA' \
  --header 'content-type: multipart/form-data' \
  --header 'x-api-key: <client_id>' \
  --form file=@ <local file path to translated .XLF content> \
  --form tenantId=<tenant_id>
```

### Successful response

```java
{
    "service": "GCS Storage",
    "response": "https://gcs...blob.core.windows.net/gcs../uploadFiles/<tenant_id>/<LocalizedAssetName>.xlf?sig=FTxG..LIw%3D&se=2023-11-29T18%3A30%3A29Z&sv=2019-02-02&sp=r&sr=b",
    "success": true
}
```

## Initiate asset locale completion in GCS

This Put API initiates asset locale completion for the specified asset and locale in GCS with manually translated asset url obtained in previous step. If a source asset is being translated in three locales, then you need to call this API three times to complete per asset per locale localization.

When you call Update asset and initiate asset complete, Globalization Content Service runs the complete asset workflow that will finally convert the translated [XLIFF](#_A_Sample_XLiff) file into the original file format and save it in the asset repository. The customer will get a URL of the target locale asset.

> **Path**: /v1/projects/{project}/tasks/{task}/assets/{asset}/locales/{locale}/complete

### Key Parameters

1. **asset name**: Unique string identifying the asset name.
1. **assetInfo**: A JSON object that describes asset properties. See the details of the assetinfo object below.
1. **locale**: Defines the target locale
1. **project:** Sets the project name.
1. **task:** Sets the task name.

### Request URL

```java
curl --request PUT \
  --url https://gcs.adobe.io/v1/projects/<projectId>/tasks/<taskId>/assets/<assetName>/locales/<targetLocale>/complete \
  --header 'Authorization: Bearer eyJhbGciOiJSUz...vCIXe11w' \
  --header 'Content-Type: application/json' \
  --header 'x-api-key: <client_id>' \
  --data '{
  "assetName": "<AssetName>",
  "tenantId": "<tenant_id>",
  "targetAssetLocale": {
    "locale": "<targetLocale>",
    "status": "TRANSLATED"
  },
  "targetAssetUrl": {
    "locale": "<targetLocale>",
    "url": "https://gcsstorageprod.blob.core.windows.net/gcs/uploadFiles/<tenant_id>/<LocalizedAssetName>.xlf?sig=5x2q..90hdbY%3D&se=2023-09-11T19%3A35%3A33Z&sv=2019-02-02&sp=r&sr=b",
    "urlType": "TRANSLATED"
  }
}'
```

### Responses

You get the following responses:

1. **200 OK**: The 200 OK status code indicates that the request has been processed successfully on the server
1. **201 Created**: The 201 Created success status code indicates that the request is successful, and it has created a resource.
1. **401 Unauthorized**: The 401 unauthorized error status response code indicates that the request was not processed because the destination resource's authentication credentials were invalid. Reauthenticating with right credentials is recommended.
1. **403 Forbidden**: The 403 Forbidden error status code indicates that the server has received the request but has refused to allow it. The client may have insufficient rights to a resource.
1. **404 Not Found**: The 404 error is returned when the server can't find the requested resource. It could be broken or dead link because of file movement.

### Understanding the successful response

Here's a successful response:

```java
{
    "tenantId": "<tenant_id>",
    "assetName": "<AsstName>",
    "targetAssetLocale": {
        "locale": "ja-JP",
        "status": "TRANSLATED"
    },
    "targetAssetUrl": {
        "locale": "ja-JP",
        "url": "https://gcsstorageprod.blob.core.windows.net/gcs/uploadFiles/<tenant_id>/<LocalizedAssetName>.xlf?sig=5x2q..90hdbY%3D&se=2023-09-11T19%3A35%3A33Z&sv=2019-02-02&sp=r&sr=b",
        "urlType": "TRANSLATED"
    },
    "workflowInstanceId": "M:61df6ed2-d31f-4790-ab43-fae711dc4c60"
}
```

### Understanding the properties

| **Property**           | **Type**            | **Description**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| :--------------------- | :------------------ | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **assetId**            | String              | A unique ID that is used by the content system to track assets if needed.                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| **assetMetadata**      | [...]               | [optional] An array of additional properties that can be added by the content system for easy asset retrieval, such as asset path. The project creation team can also use assetMetadata to pass information about the asset that can help in efficient localization.                                                                                                                                                                                                                                                    |
| **assetName**          | string              | Unique name of the asset in a task. In our example, it's "UserTest.html"                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| **mimeType**           | string              | [optional] Defines the nature and format of the asset (media type)                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| **tenantId**       | String              |  The unique ID of the tenant within a customer org.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| **orgId**              | String              | The IMS Org ID against which an org is authenticated into the Adobe I/O console.                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| **projectId**          | String              | The Id of the project containing the tasks                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| **sourceLocale**       | String              | Source locale for the task                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| **sourceUrl**          | String              | Publicly available URLs (S3 or Azure) for downloading the source asset                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| **targetAssetLocale**  | {AssetLocaleBean...} | Defines the target asset locale, includes locale, status and updatedDate. The asset information for each target asset locale is different. We have as many assets locale information as the number of locales for the assets. To retrieve assets for multiple locales make multiple API calls with asset and target locale.                                                                                                                                                                                             |
| **targetAssetUrl**     | {AssetUrlBean...}    | Publicly available URLs (S3 or Azure) for downloading the translated locale assets. This URL represents the Xliff file that includes the source locale value and target locale information. Includes information on locale, creation date, url and urlType. <br/><br/>The system will use the denormalization worker to (1) download the translated information (2) convert it to the source format, (3) upload it to the storage system (S3 or Azure) and (4) send the denormalized system back to the content system. |
| **taskId**             | String              | A unique id for the translation task.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| **workflowInstanceId** | String              | Useful when the AssetInfo bean is used as output. Unique string for the current workflow instance.                                                                                                                                                                                                                                                                                                                                                                                                                      |

> **Note**: To know more about updating an asset and initiating the asset complete process, see [Update Asset and Initiate Asset Complete API in Swagger](https://gcscore-dev-va7.stage.cloud.adobe.io/swagger-ui/index.html#/Asset%20Service/completeAssetUsingPUT).

### A Sample XLIFF File

See a sample XLIFF file.

```java
<?xml version="1.0" encoding="UTF-8"?>
<xliff version="1.2" xmlns="urn:oasis:names:tc:xliff:document:1.2" xmlns:okp="okapi-framework:xliff-extensions" xmlns:its="http://www.w3.org/2005/11/its" xmlns:itsxlf="http://www.w3.org/ns/its-xliff/" its:version="2.0">
<file original="UserTest.html" source-language="en-US" target-language="en-XX" datatype="html">
<body>
<trans-unit id="tu1" restype="x-title">
<source xml:lang="en-US">Steps - Raw Document to Filter Events</source>
<target xml:lang="en-XX">Steps - Raw Document to Filter Events</target>
</trans-unit>
<trans-unit id="tu2" restype="x-td">
<source xml:lang="en-US"></source>
<target xml:lang="en-XX"></target>
</trans-unit>
<trans-unit id="tu3" restype="x-h1">
<source xml:lang="en-US">Okapi Framework - Steps</source>
<target xml:lang="en-XX">Okapi Framework - Steps</target>
</trans-unit>
<trans-unit id="tu4" restype="x-h2">
<source xml:lang="en-US">Raw Document to Filter Events Step</source>
<target xml:lang="en-XX">Raw Document to Filter Events Step</target>
</trans-unit>
</body>
</file>
</xliff>
```

## Cancel Individual Asset API

This Put API cancels an existing asset and a locale. The cancelled asset is removed from the task list.

> **Path**: /v1/projects/{project}/tasks/{task}/assets/{asset}/locales/{locale}?tenantId={tenantId}

### Key Parameters

1. **asset**: The name of the asset to be cancelled.
2. **locale**: The locale information of asset that needs to be cancelled.
3. **tenantId**: The unique ID of the tenant within a customer org.
4. **project**: The Id of the project under which tasks are created.
5. **task:** A unique id for the translation task.

### Request URL

```java
curl --request PUT \
  --url 'https://gcs.adobe.io/v1/projects/<projectId>/tasks/<taskId>/assets/<assetName>/locales/<targetLocale>/cancel?tenantId=<tenant_id>' \
  --header 'Authorization: Bearer eyJhbGciOiJ....8MjC7kr6HOm28aw' \
  --header 'x-api-key: <client_id>'
```

### Responses

You get the following responses:

1. **200 OK**: The 200 OK status code indicates that the request has been processed successfully on the server
1. **401 Unauthorized**: The 401 unauthorized error status response code indicates that the request was not processed because the destination resource's authentication credentials were invalid. Reauthenticating with right credentials is recommended.
1. **403 Forbidden**: The 403 Forbidden error status code indicates that the server has received the request but has refused to allow it. The client may have insufficient rights to a resource.
1. **404 Not Found**: The 404 error is returned when the server can't find the requested resource. It could be broken or dead link because of file movements or entering a non-existent task name.

### Understanding the Successful Response

```java
{
    "locale": "ja-JP",
    "status": "CANCELLED",
    "updatedDate": "2023-12-13T07:23:27.643Z"
}
```

The key parameters in response are:

- **Project Id**: The Id of the project under which tasks are created
- **Asset name**: An asset for which you want to cancel the translation workflow
- **Task Id**: A unique id for the translation task
- **Locale**: The locale of the asset to be cancelled
- **Status**: CANCELLED
- **updatedDate**: 2021-09-16T04:02:32.969Z [date time of the cancellation]

> **Note**: To know more about canceling an existing asset, see [Cancel an existing asset API in Swagger](https://gcscore-dev-va7.stage.cloud.adobe.io/swagger-ui/index.html#/Asset%20Service/cancelAssetUsingPUT).

## Where do I find more information?

Follow these guides:

- Get an overview of the Globalization Content Service:
  Follow this [guide](../overview/index.md).

- Enable a partner to use the Globalization Content Service: Follow this [guide](../partner/index.md) to onboard on the Adobe I/O console and access the Globalization Content Service APIs.
