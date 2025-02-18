---
layout: default
title: Simple Search
nav_order: 3
redirect_from:
  - /guides/v2/guide_simple-search-v2.html
parent: Repository Search
grand_parent: Guides
---

<!--© 2024 Laserfiche.
See LICENSE-DOCUMENTATION and LICENSE-CODE in the project root for license information.-->

# Simple Search
**Applies to**: Repository API v2.
<br/>
<sup>[See Repository API v1](../guide_simple-search-v1/).</sup>

The simple search API can run a search query in the repository and return the results of the search immediately in the response of the call. See [this guide](../guide_search-vs-simple-search/) to view the differences between simple search and search.

In the following simple search, we want to find entries with a specific value in the _Purchase Order ID_ field.

```
POST https://api.laserfiche.com/repository/v2/Repositories/{repositoryId}/SimpleSearches
```
```json
{
"searchCommand":"{[]:[Purchase Order ID]=\"789\"}"
}
```

The response will include a listing of the entries that matched the search query. See the following sample response for the above search:

```json
{
  "@odata.context": "https://api.laserfiche.com/repository/v2/$metadata#Collection(Laserfiche.Repository.Entry)",
  "value":[
    {
      "@odata.type": "#Laserfiche.Repository.SearchResultEntry",
      "id": 1234,
      "isContainer": false,
      "isLeaf": true,
      "name": "PurchaseOrder",
      "parentId": 1,
      "fullPath": null,
      "folderPath": null,
      "creator": "Guide User",
      "creationTime": "2023-09-01T19:11:22Z",
      "lastModifiedTime": "2023-09-11T07:37:38Z",
      "entryType": "Document",
      "templateName": null,
      "templateId": 0,
      "volumeName": "DEFAULTVOLUME",
      "rowNumber": 1
    }
  ]
}
```

## Additional Information

Simple searches are limited to a maximum of **100** results. An HTTP _206 Partial Content_ status code indicates that the search results are truncated. A non-truncated search response will include an HTTP _200 OK_ status code.

Simple searches will automatically time out if the search operation exceeds **15** seconds.
A single session, distinguished by a valid OAuth access token, can only have a limited number of searches running at one time. This limit does not differentiate between search and simple search.

{: .note }

- For more information about the API limits, see [this page](../../../getting-started/guide_api-limits/).
- See the Laserfiche user guide for more information on the [Laserfiche Search Syntax](https://doc.laserfiche.com/laserfiche.documentation/11/userguide/en-us/Default.htm#../Subsystems/client_wa/Content/Search/Advanced/Template_Field.htm).
- See the Laserfiche guide on how to use the `fields` query parameter to [get field metadata with the search results](../../documents-and-folders/guide_get-folder-listing/#retrieve-field-metadata-for-each-document).
