# Chatbot - Custom Chatbot Integration

* [Overview](#overview)
* [Inbound Messages](#inbound-messages)
* [Webhooks](#webhooks)

## Overview

This document describes the requirements for development of custom chatbot for OneTalk.

With custom chatbot, business must provide an API URL which can handle inbound messages.
Business can connect any channels in its organization to a chatbot.
When a message is received by a channel connected to the chatbot, OneTalk will forward the inbound message and case information to the chatbot URL. For more information, see [Inbound Messages](#inbound-messages).

Custom chatbot can interact with the case via OneTalk's provided webhooks. For more information, see [Webhooks](#webhooks).

When creating custom chatbot in OneTalk dashboard, a secret key and webhook URL will be generated once. The webhook URL is used to interact with a case. The secret key is required when sending request to OneTalk webhooks. The secret key will also be provided in the HTTP header whenever OneTalk forward an inbound message to the chatbot, check the value to verify if the request is valid.

## Inbound Messages

Inbound messages received by a channel will be forwarded to the custom chatbot's API URL. Custom chatbot must be able to process the API requests and return response with HTTP status 200 on success. In case the chatbot fail to return HTTP status 200, OneTalk will try to send the API request for a limited number of attempts before dropping request for that message.

Specifications for the API request:

### HTTP Method

```
POST
```

### HTTP Headers

| Header       | Description
|:-------------|:------------------------------------------------------------------------
| X-Secret-Key | Secret Key from OneTalk dashboard, can be used to validate the request.

### Request Body

The request body will always be a JSON object.

| Field       | Type                              | Description
|:------------|:----------------------------------|:---------------------------------------------------------------
| caseID      | string                            | The case ID which the message is part of.
| channelType | string                            | The channel type ("whatsapp", "whatsappba", "telegram", etc.).
| sender      | [InboundSender](#inboundsender)   | The sender's information.
| message     | [InboundMessage](#inboundmessage) | The inbound message's details.

Example:

```json
{
  "caseID": "FBED34A1EF",
  "channelType": "whatsappba",
  "sender": {...},
  "message": {...}
}
```

#### InboundSender

| Field       | Type    | Description
|:------------|:--------|:--------------------------------------------------------------------------------
| phone       | string  | The sender's phone number (only if channel type is "whatsapp" or "whatsappba").
| contactName | string  | The contact name.

Example:

```json
{
  "phone": "62851234567890",
  "contactName": "John Doe"
}
```

#### InboundMessage

| Field      | Type                                                  | Description
|:-----------|:------------------------------------------------------|:---------------------------------------------------------------
| id         | string                                                | The message ID.
| type       | string                                                | The message type.
| text       | [InboundMessageText](#inboundmessagetext)             | Details for message type "text".
| document   | [InboundMessageDocument](#inboundmessagedocument)     | Details for message type "document".
| image      | [InboundMessageMedia](#inboundmessagemedia)           | Details for message type "image".
| video      | [InboundMessageMedia](#inboundmessagemedia)           | Details for message type "video".
| location   | [InboundMessageLocation](#inboundmessagelocation)     | Details for message type "location".
| whatsappba | [InboundMessageWABA](#inboundmessagewaba) | Additional details for channel type "whatsappba", if provided.

Examples:

```json
{
  "id": "wamid.HBgNNjI...",
  "type": "text",
  "text": {
    "body": "Hello world"
  }
}

{
  "id": "wamid.HBgNNjI...",
  "type": "document",
  "document": {
    "url": "https://...",
    "filename": "Receipt.pdf",
    "caption": ""
  }
}

{
  "id": "wamid.HBgNNjI...",
  "type": "image",
  "image": {
    "url": "https://...",
    "caption": ""
  }
}

{
  "id": "wamid.HBgNNjI...",
  "type": "video",
  "image": {
    "url": "https://...",
    "caption": ""
  }
}

{
  "id": "wamid.HBgNNjI4NTE2MzUyMTI5MRUCABIYIDA3OEM0MUFDRTkwMTNERkU5MjYxMTU0NjlGNkU1RTM2AA==",
  "type": "location",
  "location": {
    "latitude": -6.1972252057581,
    "longitude": 106.76131798074,
    "address": "...",
    "name": ""
  }
}

{
  "id": "wamid.HBgNNjI...",
  "type": "text",
  "text": {
    "body": "Reschedule"
  },
  "whatsappba": {
    "type": "button",
    "button": {
      "payload": "reschedule",
      "text": "Reschedule"
    }
  }
}

{
  "id": "wamid.HBgNNjI...",
  "type": "text",
  "text": {
    "body": "Talk to agent"
  },
  "whatsappba": {
    "type": "interactive",
    "interactive": {
      "type": "button_reply",
      "button_reply": {
        "id": "/agent",
        "title": "Talk to agent"
      }
    }
  }
}
```

#### InboundMessageText

| Field | Type    | Description
|:------|:--------|:--------------------------
| body  | string  | Body of the text message.

```json
{
  "body": "Hello world"
}
```

#### InboundMessageDocument

| Field    | Type    | Description
|:---------|:--------|:--------------------------
| url      | string  | The document's file URL.
| filename | string  | The document's file name.
| caption  | string  | The document's caption.

Example:

```json
{
  "url": "https://...",
  "filename": "Receipt.pdf",
  "caption": ""
}
```

#### InboundMessageMedia

| Field    | Type    | Description
|:---------|:--------|:-------------------------------
| url      | string  | The image or video's file URL.
| caption  | string  | The image or video's caption.

Example:

```json
{
  "url": "https://...",
  "caption": ""
}
```

#### InboundMessageLocation

| Field     | Type    | Description
|:----------|:--------|:-------------------------
| latitude  | float   | Location latitude.
| longitude | float   | Location longitude.
| address   | string  | Address of the location.
| name      | string  | Name of the location.

Example:

```json
{
  "latitude": -6.1972252057581,
  "longitude": 106.76131798074,
  "address": "...",
  "name": ""
}
```

#### InboundMessageWABA

| Field       | Type                                                            | Description
|:------------|:----------------------------------------------------------------|:----------------------------------------------
| type        | string                                                          | Message type from WhatsApp. Values: "button", "interactive".
| button      | [InboundMessageWABAButton](#inboundmessagewababutton)           | Details for message type "button".
| interactive | [InboundMessageWABAInteractive](#inboundmessagewabainteractive) | Details for message type "interactive".

Examples:

```json
{
  "type": "button",
  "button": {
    "payload": "reshedule",
    "text": "Reschedule"
  }
}

{
  "type": "interactive",
  "interactive": {...}
}
```

#### InboundMessageWABAButton

| Field   | Type    | Description
|:--------|:--------|:---------------------------------------------------------------------------------
| payload | string  | Payload of the button that a customer clicked as part of an interactive message.
| text    | string  | Button text.

Example:

```json
{
  "payload": "reshedule",
  "text": "Reschedule"
}
```

#### InboundMessageWABAInteractive

| Field        | Type                                                            | Description
|:-------------|:----------------------------------------------------------------|:----------------------------------------------
| type         | string                                                          | The reply type. Values: "button_reply", "list_reply".
| button_reply | [InboundMessageWABAButtonReply](#inboundmessagewababuttonreply) | Details for reply type "button_reply", sent when a customer clicks a button.
| list_reply   | [InboundMessageWABAListReply](#inboundmessagewabalistreply)     | Details for reply type "list_reply", sent when a customer selects an item from a list.

Examples:

```json
{
  "type": "button_reply",
  "button_reply": {
    "id": "/agent",
    "title": "Talk to agent"
  }
}

{
  "type": "list_reply",
  "list_reply": {
    "id": "/agent 0",
    "title": "More options",
    "description": "For more topic options"
  }
}
```

#### InboundMessageWABAButtonReply

| Field       | Type    | Description
|:------------|:--------|:-------------------------------------
| id          | string  | Unique ID of the button.
| title       | string  | Title of the button.

Example:

```json
{
  "id": "/agent",
  "title": "Talk to agent"
}
```

#### InboundMessageWABAListReply

| Field       | Type    | Description
|:------------|:--------|:-------------------------------------
| id          | string  | Unique ID of the selected list item.
| title       | string  | Title of the selected list item.
| description | string  | Description of the selected row.

Example:

```json
{
  "id": "/agent 0",
  "title": "More options",
  "description": "For more topic options"
}
```

## Webhooks

To manage a case or reply to a message, chatbot can send a request to the dedicated webhook URL provided by OneTalk.

There are 3 events supported:
- `messages`: Handle outbound messages.
- `handover_case`: Handover case from chatbot to agent.
- `close_case`: Close the case.
- `update_case`: Update the case's details.
- `update_contact`: Update the case's associated contact.

### HTTP Method

```
POST
```

### HTTP Headers

| Header     | Description
|:-----------|:--------------------------------------------------------------------------
| Secret-Key | The chatbot's secret key generated by OneTalk, required on every request.

### Request Body

The request body must be JSON.

| Field         | Type                                          | Description
|:--------------|:----------------------------------------------|:-----------------------------------------------------------------
| caseID        | string                                        | The case ID.
| eventType     | string                                        | The event type ("messages", "handover_case" "close_case", or "update_contact").
| messages      | array of [OutboundMessage](#outboundmessage)  | `Optional` The list of outbound messages to be sent as chatbot's response.
| handoverCase  | [WebhookHandoverCase](#webhookhandovercase)   | `Optional` Details for the handover.
| closeCase     | [WebhookCloseCase](#webhookclosecase)         | `Optional` Details for closing the case.
| updateCase    | [WebhookUpdateCase](#webhookupdatecase)       | `Optional` Details for updating the case.
| updateContact | [WebhookUpdateContact](#webhookupdatecontact) | `Optional` Details for updating the contact.

Examples:

```json
{
  "caseID": "FBED34A1EF",
  "eventType": "messages",
  "messages": [{...}]
}

{
  "caseID": "FBED34A1EF",
  "eventType": "handover_case",
  "handoverCase": {...}
}

{
  "caseID": "FBED34A1EF",
  "eventType": "close_case",
  "closeCase": {...}
}

{
  "caseID": "FBED34A1EF",
  "eventType": "update_case",
  "updateCase": {...}
}

{
  "caseID": "FBED34A1EF",
  "eventType": "update_contact",
  "updateContact": {...}
}
```

#### OutboundMessage

OneTalk supports 4 basic message types:
- `text`
- `document`
- `image`
- `video`

OneTalk also supports channel-specific message types:
- `whatsappba`: For cases from WABA channel.

| Field      | Type                                                | Description
|:-----------|:----------------------------------------------------|:---------------------------------------------------------------
| type       | string                                              | The message type.
| text       | [OutboundMessageText](#outboundmessagetext)         | `Optional` Details for message type "text".
| document   | [OutboundMessageDocument](#outboundmessagedocument) | `Optional` Details for message type "document".
| image      | [OutboundMessageMedia](#outboundmessagemedia)       | `Optional` Details for message type "image".
| video      | [OutboundMessageMedia](#outboundmessagemedia)       | `Optional` Details for message type "video".
| whatsappba | [OutboundMessageWABA](#outboundmessagewaba)         | `Optional` Details for message type "whatsappba", availalbe for channel type "whatsappba".

Examples:

```json
{
  "type": "text",
  "text": {
    "body": "Hi, what's your name?"
  }
}

{
  "type": "document",
  "document": {
    "url": "https://...",
    "filename": "Receipt.pdf",
    "caption": ""
  }
}

{
  "type": "image",
  "image": {
    "url": "https://...",
    "caption": "..."
  }
}

{
  "type": "video",
  "image": {
    "url": "https://...",
    "caption": "..."
  }
}

{
  "type": "whatsappba",
  "whatsappba": {
    "type": "interactive",
    "interactive": {
      "type": "button",
      "header": {...},
      "body": {
        "text": "..."
      },
      "footer": {...},
      "buttonAction": {
        "buttons": [{...}]
      }
    }
  }
}

{
  "type": "whatsappba",
  "whatsappba": {
    "type": "interactive",
    "interactive": {
      "type": "list",
      "body": {
        "text": "..."
      },
      "listAction": {
        "button": "...",
        "sections": [{...}]
      }
    }
  }
}
```

#### OutboundMessageText

| Field | Type    | Description
|:------|:--------|:--------------------------
| body  | string  | Body of the text message.

```json
{
  "body": "Hi, what's your name?"
}
```

#### OutboundMessageDocument

| Field    | Type    | Description
|:---------|:--------|:--------------------------
| url      | string  | The document's file URL.
| filename | string  | The document's file name.
| caption  | string  | The document's caption.

Example:

```json
{
  "url": "https://...",
  "filename": "Receipt.pdf",
  "caption": "..."
}
```

#### OutboundMessageMedia

| Field    | Type    | Description
|:---------|:--------|:-------------------------------
| url      | string  | The image or video's file URL.
| caption  | string  | The image or video's caption.

Example:

```json
{
  "url": "https://...",
  "caption": "..."
}
```

#### OutboundMessageWABA

| Field       | Type                                                              | Description
|:------------|:------------------------------------------------------------------|:--------------------------------------------------
| type        | string                                                            | Message type from WhatsApp. Values: "interactive".
| interactive | [OutboundMessageWABAInteractive](#outboundmessagewabainteractive) | Details for message type "interactive".

Examples:

```json
{
  "type": "interactive",
  "interactive": {
    "type": "button",
    "header": {...},
    "body": {
      "text": "..."
    },
    "footer": {...},
    "buttonAction": {
      "buttons": [{...}]
    }
  }
}

{
  "type": "interactive",
  "interactive": {
    "type": "list",
    "body": {
      "text": "..."
    },
    "listAction": {
      "button": "...",
      "sections": [{...}]
    }
  }
}
```

#### OutboundMessageWABAInteractive

There are 2 supported interactive types:
- `button`: For Reply Buttons.
- `list`: For List Messages.

| Field        | Type                                                                                      | Description
|:-------------|:------------------------------------------------------------------------------------------|:-------------------------
| type         | string                                                                                    | Type of interactive message. Values: "button", "list".
| header       | [OutboundMessageWABAInteractiveHeader](#outboundmessagewabainteractiveheader)             | `Optional` Header of the message.
| body         | [OutboundMessageWABAInteractiveBody](#outboundmessagewabainteractivebody)                 | Body of the message.
| footer       | [OutboundMessageWABAInteractiveFooter](#outboundmessagewabainteractivefooter)             | `Optional` Footer of the message.
| buttonAction | [OutboundMessageWABAInteractiveButtonAction](#outboundmessagewabainteractivebuttonaction) | `Optional` Action for interactive type "button".
| listAction   | [OutboundMessageWABAInteractiveListAction](#outboundmessagewabainteractivelistaction)     | `Optional` Action for interactive type "list".

Examples:

```json
{
  "type": "button",
  "header": {...},
  "body": {
    "text": "..."
  },
  "footer": {...},
  "buttonAction": {
    "buttons": [{...}]
  }
}

{
  "type": "list",
  "body": {
    "text": "..."
  },
  "listAction": {
    "button": "...",
    "sections": [{...}]
  }
}
```

#### OutboundMessageWABAInteractiveHeader

| Field      | Type                                                                              | Description
|:-----------|:----------------------------------------------------------------------------------|:-----------------------------------
| type       | string                                                                            | The header type. Values: "text", "document", "image", "video".
| text       | string                                                                            | `Optional` The header's text,  if header type is "text".
| document   | [OutboundMessageWABAInteractiveDocument](#outboundmessagewabainteractivedocument) | `Optional` Details for header type "document".
| image      | [OutboundMessageWABAInteractiveMedia](#outboundmessagewabainteractivemedia)       | `Optional` Details for message type "image".
| video      | [OutboundMessageWABAInteractiveMedia](#outboundmessagewabainteractivemedia)       | `Optional` Details for message type "video".

Examples:

```json
{
  "type": "text",
  "text": "Welcome!"
}

{
  "type": "document",
  "document": {
    "link": "https://...",
    "filename": "Receipt.pdf"
  }
}

{
  "type": "image",
  "image": {
    "link": "https://..."
  }
}

{
  "type": "video",
  "image": {
    "link": "https://..."
  }
}
```

#### OutboundMessageWABAInteractiveDocument

| Field    | Type    | Description
|:---------|:--------|:--------------------------
| link     | string  | The document's file URL.
| filename | string  | The document's file name.


#### OutboundMessageWABAInteractiveMedia

| Field    | Type    | Description
|:---------|:--------|:-------------------------------
| link     | string  | The image or video's file URL.

#### OutboundMessageWABAInteractiveBody

| Field   | Type    | Description
|:--------|:--------|:-----------------
| text    | string  | The body's text.

Example:

```json
{
  "text": "How can we help you?"
}
```

#### OutboundMessageWABAInteractiveFooter

| Field   | Type    | Description
|:--------|:--------|:-------------------
| text    | string  | The footer's text.

Example:

```json
{
  "text": "This is an automatic reply"
}
```

#### OutboundMessageWABAInteractiveButtonAction

| Field   | Type                                                                                   | Description
|:--------|:---------------------------------------------------------------------------------------|:-------------------------------
| buttons | array of [OutboundMessageWABAInteractiveButton](#outboundmessagewabainteractivebutton) | The list of buttons, maximum 3 buttons.

Example:

```json
{
  "buttons": [{...}]
}
```

#### OutboundMessageWABAInteractiveButton

| Field | Type                                                                                    | Description
|:------|:----------------------------------------------------------------------------------------|:-----------------------------
| type  | string                                                                                  | The button type. The only supported type is "reply".
| reply | [OutboundMessageWABAInteractiveReplyButton](#outboundmessagewabainteractivereplybutton) | Details of the Reply Button.

Example:

```json
{
  "type": "reply",
  "reply": {
    "id": "/agent",
    "title": "Talk to agent"
  }
}
```

#### OutboundMessageWABAInteractiveReplyButton

| Field | Type    | Description
|:------|:--------|:-----------------------------------------------------------------------------------------------------------------
| id    | string  | Unique identifier for the button. Maximum length: 256 characters.
| title | string  | The button's title, must be unique within the message. Emojis are supported, markdown is not. Maximum length: 20 characters.

#### OutboundMessageWABAInteractiveListAction

| Field    | Type                                                                                     | Description
|:---------|:-----------------------------------------------------------------------------------------|:-----------------------------
| button   | string                                                                                   | Button to show the list. Maximum length: 20 characters.
| sections | array of [OutboundMessageWABAInteractiveSection](#outboundmessagewabainteractivesection) | List of sections, maximum 10 sections.

Example:

```json
{
  "button": "Select topic",
  "sections": [{...}]
}
```

#### OutboundMessageWABAInteractiveSection

| Field | Type                                                                                           | Description
|:------|:-----------------------------------------------------------------------------------------------|:--------------------------
| title | string                                                                                         | `Optional` Title of the section. Required if the message has more than one section.
| rows  | array of [OutboundMessageWABAInteractiveSectionRow](#outboundmessagewabainteractivesectionrow) | The list of rows. Maximum 10 rows across all sections.

Example:

```json
{
  "title": "Frequently selected",
  "rows": [{...}]
}
```

#### OutboundMessageWABAInteractiveSectionRow

| Field       | Type    | Description
|:------------|:--------|:------------------------------------------------------------------
| id          | string  | Unique identifier for the row. Maximum length: 200 characters.
| title       | string  | Title of the row. Maximum length: 24 characters.
| description | string  | `Optional` Description of the row. Maximum length: 72 characters.

```json
{
  "id": "/agent 0",
  "title": "More options",
  "description": "For more topic options"
}
```

#### WebhookHandoverCase

If the parameter `topicID` is not set or value is 0 and the case is not assigned to any topic yet,
then the case will be automatically assigned to the channel's topic.
In case the channel has multiple topics, then a message containing topic options will be sent to the customer.

If the parameter `agentEmail` is set, the selected agent must be assigned to the case's assigned topic.

| Field       | Type     | Description
|:------------|:---------|:------------------------------------------------------
| topicID     | integer  | `Optional` ID of the topic to handover to. 
| agentEmail  | string   | `Optional` Email address of the agent to handover to.

Example:

```json
{
  "topicID": 123,
  "agentEmail": ""
}
```

#### WebhookCloseCase

| Field              | Type    | Description
|:-------------------|:--------|:-------------------------------------------------------------------------------
| sendClosingMessage | boolean | `Optional` If true, closing message will be sent to the customer if available.

Example:

```json
{
  "sendClosingMessage": false
}
```

#### WebhookUpdateCase

Only provided fields will be updated.

| Field        | Type               | Description
|:-------------|:-------------------|:-------------------------------------------------------------------------------
| agentRemark  | string             | `Optional` Agent remark for the case, may be empty.
| isJunk       | boolean            | `Optional` If the case is junk.

Example:

```json
{
  "agentRemark": "...",
  "isJunk": true
}
```

#### WebhookUpdateContact

Only provided fields will be updated.

| Field        | Type               | Description
|:-------------|:-------------------|:-------------------------------------------------------------------------------
| fullName     | string             | `Optional` The contact's full name.
| alias        | string             | `Optional` The contact's alias, may be empty.
| companyName  | string             | `Optional` The contact's company name, may be empty.
| jobTitle     | string             | `Optional` The contact's job title, may be empty.
| customFields | map[string:string] | `Optional` The custom fields to be updated.
| - key        | string             | The custom field code to be updated.
| - value      | string             | Value of the custom field, may be empty depending on the custom field's setup.

Example:

```json
{
  "fullName": "John Doe",
  "alias": "John",
  "companyName": "PT Tap Talk Teknologi",
  "jobTitle": "", // Set the value as empty string
  "customFields": {
    // "field_code": "value"
    "company_address": "Business Park Kebon Jeruk blok C2-3\nMeruya Utara...", // Field type: Multiple Lines
    "company_city": "Jakarta", // Field type: Single Line
    "birthday": "2000-12-31", // Field type: Date
    "products": "[\"OneTalk\",\"SendTalk\",\"PowerTalk\"]", // Field type: Multiple Select (JSON-encoded array)
    "account_manager": "" // Set the value as empty
  }
}

{
  "fullName": "John Doe",
  "alias": "",
  "customFields": {
    "membership_id": "202401234567890",
    ...
  }
}
```
