# Ticketing System - Custom Provider Integration

* [Overview](#overview)
* [API Specifications](#api-specifications)
* [References](#references)
* [Object Models](#object-models)

## Overview

This document describes the requirements for custom provider integration for Ticketing System.

With custom integration, business must provide an API URL which OneTalk will use to communicate with the custom provider via API. The API must be able to serve different response based on a request's parameter.

## API Specifications

The API request will always have the following specifications:

**HTTP Method**

```
POST
```

**HTTP Headers**

| Header       | Description
|:-------------|:------------------------------------------------------------------------
| X-Secret-Key | Secret Key from OneTalk dashboard, can be used to validate the request.

**Request Body**

The request body will always be a JSON object.

| Field  | Type   | Description
|:-------|:-------|:---------------------------------------------------------------------
| action | string | The requested action (e.g.: "get_create_ticket_url", "get_ticket_status_list_by_contact", etc.)
| data   | object | The request parameters based on the request type.

Example:

```json
{
  "action": "{{action}}",
  "data": {
    // Any parameters based on `action` value.
  }
}
```

**Response Status Code**

* For successful requests, the API must always return HTTP status 200.
* For failed requests, the API may return HTTP status 4xx or 5xx.

**Response Body**

* For successful requests, the response body must be a JSON object based on the request action. See [References](#references) for more details.
* For failed requests, if the response body is a string, it may be used as an error message.

## References

* [Get Create Ticket URL](#get-create-ticket-url)
* [Get Ticket Status List by Case](#get-ticket-status-list-by-case)
* [Get Ticket Status List by Contact](#get-ticket-status-list-by-contact)
* [Get Ticket History URL](#get-ticket-history-url)

### Get Create Ticket URL

Get URL for creating a ticket. The custom provider must provide a page which agent can use to create a ticket.

OneTalk will send details of the case which should be associated with the created ticket, and custom provider can generate a dynamic URL based on the case and contact's details. It is recommended to store the details locally and return a temporary ID in the created URL.

**Request Data**

| Field      | Type                | Description
|:-----------|:--------------------|:-----------------------------------------------
| case       | [Case](#case)       | The case's details.
| contact    | [Contact](#contact) | The contact's information.
| agentEmail | string              | Email address of the agent sending the request.

**Request Example**

```json
{
  "action": "get_create_ticket_url",
  "data": {
    "case": {...},
    "contact": {...},
    "agentEmail": "luna@example.com"
  }
}
```

**Response Data**

| Field | Type   | Description
|:------|:-------|:----------------------------------------------
| url   | string | URL to the page for creating ticket.

**Response Example**

```json
{
  "url": "https://example.com/ticketing/create?case_id=FBED34A1EF"
}
```

### Get Ticket Status List by Case

Get ticket status list for a case. OneTalk will send details of the case which tickets must be returned for.

Custom provider must return list of fields to be displayed and list of tickets in the form of key-value pair, where key represents the field name and value represents the field value.

**Request Data**

| Field      | Type                | Description
|:-----------|:--------------------|:-----------------------------------------------
| case       | [Case](#case)       | The case's details.
| contact    | [Contact](#contact) | The contact's information.
| agentEmail | string              | Email address of the agent sending the request.
| search     | string              | `Optional` The search term.
| pageNumber | integer             | The page number.
| pageSize   | integer             | Number of items per page.

**Request Example**

```json
{
  "action": "get_ticket_status_list_by_case",
  "data": {
    "case": {...},
    "contact": {...},
    "agentEmail": "luna@example.com",
    "search": "",
    "pageNumber": 1,
    "pageSize": 20
  }
}
```

**Response Data**

| Field      | Type                        | Description
|:-----------|:----------------------------|:----------------------------------------------------------------------------------------------
| fields     | array of string             | The list of fields to be displayed. The fields must be present in each item.
| items      | array of map[string:string] | The list of tickets. A ticket consists of key-value pairs containing the ticket's information.
| - key      | string                      | Name of the field.
| - value    | string                      | Value of the field.
| hasMore    | boolean                     | If there are more items in pagination.
| totalItems | integer                     | Total number of items in pagination.
| totalPages | integer                     | Total number of pages for the requested page size.

> Make sure to calculate the `totalItems` and `totalPages` correctly.

**Response Example**

```json
{
  "fields": [ "id", "submit date", "status" ],
  "items": [
    {
      "id": "1",
      "submit date": "today",
      "status": "open"
    },
    {
      "id": "2",
      "submit date": "10 Jun 2025",
      "status": "closed"
    },
    ...
  ],
  "hasMore": false,
  "totalItems": 3,
  "totalPages": 1
}
```

### Get Ticket Status List by Contact

Get ticket status list for a contact. OneTalk will send details of the contact which tickets must be returned for.

Custom provider must return list of fields to be displayed and list of tickets in the form of key-value pair, where key represents the field name and value represents the field value.

**Request Data**

| Field      | Type                | Description
|:-----------|:--------------------|:-----------------------------------------------
| contact    | [Contact](#contact) | The contact's information.
| agentEmail | string              | Email address of the agent sending the request.
| search     | string              | `Optional` The search term.
| pageNumber | integer             | The page number.
| pageSize   | integer             | Number of items per page.

**Request Example**

```json
{
  "action": "get_ticket_status_list_by_contact",
  "data": {
    "contact": {...},
    "agentEmail": "luna@example.com",
    "search": "",
    "pageNumber": 1,
    "pageSize": 20
  }
}
```

**Response Data**

| Field      | Type                        | Description
|:-----------|:----------------------------|:----------------------------------------------------------------------------------------------
| fields     | array of string             | The list of fields to be displayed. The fields must be present in each item.
| items      | array of map[string:string] | The list of tickets. A ticket consists of key-value pairs containing the ticket's information.
| - key      | string                      | Name of the field.
| - value    | string                      | Value of the field.
| hasMore    | boolean                     | If there are more items in pagination.
| totalItems | integer                     | Total number of items in pagination.
| totalPages | integer                     | Total number of pages for the requested page size.

> Make sure to calculate the `totalItems` and `totalPages` correctly.

**Response Example**

```json
{
  "fields": [ "id", "submit date", "status" ],
  "items": [
    {
      "id": "1",
      "submit date": "today",
      "status": "open"
    },
    {
      "id": "2",
      "submit date": "10 Jun 2025",
      "status": "closed"
    },
    ...
  ],
  "hasMore": false,
  "totalItems": 3,
  "totalPages": 1
}
```

### Get Ticket History URL

Get URL for showing ticket history. The custom provider must provide a page which agent can use to view ticket history.

Custom provider can generate a dynamic URL based on the agent requesting the URL.

**Request Data**

| Field      | Type                | Description
|:-----------|:--------------------|:-----------------------------------------------
| agentEmail | string              | Email address of the agent sending the request.

**Request Example**

```json
{
  "action": "get_ticket_history_url",
  "data": {
    "agentEmail": "luna@example.com"
  }
}
```

**Response Data**

| Field | Type   | Description
|:------|:-------|:----------------------------------------------
| url   | string | URL to the ticket history page.

**Response Example**

```json
{
  "url": "https://example.com/ticket_history?agent=luna%40example.com"
}
```

## Object Models

* [Case](#case)
* [CaseReferrer](#casereferrer)
* [Contact](#contact)
* [ContactCustomField](#contactcustomfield)

### Case

| Field                      | Type                          | Description
|:---------------------------|:------------------------------|:----------------------------------------------------------------------------
| id                         | string                        | The case ID.
| channelType                | string                        | Channel type where the case is created from ("whatsapp", "whatsappba", "telegram", etc.).
| channelID                  | string                        | The channel ID.
| channelName                | string                        | The channel name.
| topicID                    | int32                         | The topic ID.
| topicName                  | string                        | The topic name.
| assigneeType               | string                        | The case's current assignee type ("agent" or "chatbot").
| chatbotID                  | string                        | ID of the chatbot currently handling the case.
| chatbotName                | string                        | Name of the chatbot currently handling the case.
| agentEmail                 | string                        | Email address of the agent handling the case.
| agentFullName              | string                        | Full name of the agent handling the case.
| agentAlias                 | string                        | Alias of the agent handling the case.
| firstMessage               | string                        | The case's first message.
| firstResponseTime          | int64                         | The time of first response from agent, in Unix milliseconds.
| firstResponseAgentEmail    | string                        | Email address of the agent who responded first.
| firstResponseAgentFullName | string                        | Full name of the agent who responded first.
| firstResponseAgentAlias    | string                        | Alias of the agent who responded first.
| createdTime                | int64                         | The time the case was created, in Unix milliseconds.
| counterStartTime           | int64                         | The time to start counting duration from, in Unix milliseconds.
| isJunk                     | boolean                       | If the case is marked as junk.
| agentRemark                | string                        | Remark by agent for the case.
| labels                     | array of string               | The case's assigned labels.
| referrer                   | [CaseReferrer](#casereferrer) | `Optional` The case's referrer & UTM, if any.

**Example:**

```json
{
  "id": "FBED34A1EF",
  "channelType": "whatsappba",
  "channelID": "2108211854",
  "channelName": "TapTalk.io",
  "topicID": 96,
  "topicName": "General",
  "assigneeType": "agent",
  "chatbotID": "",
  "chatbotName": "",
  "agentEmail": "luna@example.com",
  "agentFullName": "Luna",
  "agentAlias": "",
  "firstMessage": "Hi, I need help with my order",
  "firstResponseTime": 0,
  "firstResponseAgentEmail": "",
  "firstResponseAgentFullName": "",
  "firstResponseAgentAlias": "",
  "createdTime": 1746632763621,
  "counterStartTime": 1746728841663,
  "isJunk": false,
  "agentRemark": "",
  "labels": [ "complaint" ],
  "referrer": {...}
}
```

### CaseReferrer

| Field       | Type   | Description
|:------------|:-------|:-------------------
| fromURL     | string | The referrer URL.
| utmSource   | string | The UTM source.
| utmMedium   | string | The UTM medium.
| utmCampaign | string | The UTM campaign.
| utmTerm     | string | The UTM term.
| utmContent  | string | The UTM content.

**Example:**

```json
{
  "fromURL": "https://taptalk.io/onetalk/",
  "utmSource": "Google",
  "utmMedium": "Search Ads",
  "utmCampaign": "OneTalk Promo 2024",
  "utmTerm": "",
  "utmContent": ""
}
```

### Contact

| Field          | Type                                                  | Description
|:---------------|:------------------------------------------------------|:--------------------------------------------------------------
| id             | string                                                | The contact ID.
| fullName       | string                                                | The contact's full name.
| alias          | string                                                | The contact's alias.
| email          | string                                                | The email address.
| phone          | string                                                | The phone number.
| customerUserID | string                                                | The contact's customer user ID, if any.
| companyName    | string                                                | The contact's company name.
| jobTitle       | string                                                | The contact's job title.
| agentRemark    | string                                                | Remark by agent for the contact.
| customFields   | map[string:[ContactCustomField](#contactcustomfield)] | `Optional` The contact's information in custom fields, if any.
| - key          | string                                                | The custom field code.
| - value        | [ContactCustomField](#contactcustomfield)            | Value of the custom field.

**Example:**

```json
{
  "id": "e3a88e41-7db9-411b-bb9e-b3400a586e42",
  "fullName": "John Doe",
  "alias": "John",
  "email": "",
  "phone": "62851234567890",
  "customerUserID": "",
  "companyName": "PT Tap Talk Teknologi",
  "jobTitle": "",
  "agentRemark": "VIP customer",
  "customFields": {
    "birthday": {
      "fieldType": "date",
      "value": "2000-12-31"
    },
    "vip_level": {
      "fieldType": "number",
      "value": "1"
    },
    "pic_name": {
      "fieldType": "single_line",
      "value": "Luna"
    },
    ...
  }
}
```

### ContactCustomField

| Field     | Type   | Description
|:----------|:-------|:-----------------------------------------------------------------------------------------------------------
| fieldType | string | Type of the custom field ("single_line", "multiple_line", "dropdown", "multiple_select", "date", "number").
| value     | string | Value of the custom field.

**Example:**

```json
{
  "fieldType": "date",
  "value": "2000-12-31"
}

{
  "fieldType": "number",
  "value": "1"
}

{
  "fieldType": "single_line",
  "value": "Luna"
}
```
