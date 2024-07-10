---
layout: default
title: Using Laserfiche API from Web Request Rules
nav_order: 7
has_children: false
parent: Guides
---

<!--© 2024 Laserfiche.
See LICENSE-DOCUMENTATION and LICENSE-CODE in the project root for license information.-->

# Using Laserfiche API from Web Request Rules in a Laserfiche cloud account

Laserfiche process automation [Web Request Rules](https://doc.laserfiche.com/laserfiche.documentation/en-us/Default.htm#../Subsystems/ProcessAutomation/Content/Resources/Rules/web-request-rule.htm?TocPath=Process%2520Automation%257CRules%257C_____10) can be used to enable Workflows and Business Processes to interact with HTTP based API, also known as RESTful API.

This guide provides a ready to use examples of Web Request rules that use Laserfiche [Repository API](../../api/guide_overview-of-the-laserfiche-api/) to interact with documents in a Laserfiche cloud Repository.

{: .note }
**Hint:** This technique can be used to interact with Laserfiche resources can belong to a different account.

## 1. Create a Service Application in the Laserfiche Developer Console

- You'll need to select an existing Service Principal account, or create a new one, and then generate a Service Principal Key (record the key string, you'll need it later). Review our [dedicated guide](./../../api/authentication/guide_service-principals/) on this topic for more details.

- Create a Service Application to represent the integration with Web Service Connection. Follow instructions [here](./../../api/authentication/guide_oauth-service/) on creating a Service App in the Laserfiche Developer Console. It is important to follow the instructions for using a **long-lasting Authorization Key**.

{: .note }
**Note:** Service Principal account must be granted the roles needed to access to the resources to be exposed via Laserfiche API.

## 2. Create a Web Service Connection

Create a Web Service Connection by importing [Laserfiche API US Cloud - Web Service Connection.dsi](./assets/Laserfiche%20API%20US%20Cloud%20-%20Web%20Service%20Connection.dsi)

![](./assets/Web%20Service%20Connections.jpg)

{: .note }
**Note:** Enter the long-lasting _authorizationKey_ generated for the your Service Application as the Authentication Bearer value.

![](./assets/Laserfiche%20API%20US%20Cloud%20-%20Web%20Service%20Connection.jpg)

## 2. Create a Web Request Rule and configure its Web Service Connection

Create a Web Service Rule(s) and configure their Web Service Connection to point to the previously created Web Service Connection for your 'Laserfiche API US Cloud' account.
You can create Web Service Rule by importing the following definition files:

- [Laserfiche Repository API - Entry Import - Web Request Rule.bri](./assets/Laserfiche%20Repository%20API%20-%20Entry%20Import%20-%20Web%20Request%20Rule.bri)

- TODO

![](./assets/aserfiche%20Repository%20API%20-%20Entry%20Import%20-%20Web%20Request%20Rule.jpg)

## Test

TODO
