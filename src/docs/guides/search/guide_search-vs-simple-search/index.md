---
layout: default
title: Search vs. Simple Search
nav_order: 1
redirect_from:
  - /guides/v2/guide_search-vs-simple-search-v2.html
parent: Repository Search
grand_parent: Guides
---

<!--© 2024 Laserfiche.
See LICENSE-DOCUMENTATION and LICENSE-CODE in the project root for license information.-->

# Search vs. Simple Search
**Applies to**: Repository API v2.
<br/>
<sup>[See Repository API v1](../guide_search-vs-simple-search-v1/).</sup>

Laserfiche API V2 provides two search APIs, each optimized for different use cases:

- **Simple Search:** that will return the results of the search in the immediate response body.
- **Search:** that performs the search in an _asynchronous_ style, i.e. using the [long operation](../../../getting-started/guide_long-operations/) pattern.

## Simple Search

The [Simple Search](../guide_simple-search/) API is a single API call that will return the results of the search in the immediate response body of the call. It supports all Laserfiche search syntax queries, but will truncate the number of results and the information returned in the results. You should use a simple search when:

- You do not expect a large number of search results, for example, when searching to find a specific entry in the repository by searching on a Laserfiche field value with a specific value that is a unique ID.
- You do not need context hits, for example, when running searches on criteria like field values.

## Search

The [Search](../guide_search/) API exposes the full search capabilities of Laserfiche. You should use the Search API when:

- You expect many results from your search, for example, when the search criteria includes date ranges that may return an unbounded number of results.
- The search criteria involves finding content within documents, for example, when searching on document text and you want to retrieve context hits for each result of the search.
