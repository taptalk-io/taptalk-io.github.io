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

**Example:**

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

**Example:**

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

**Examples:**

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

**Example:**

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

**Example:**

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

**Example:**

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

**Examples:**

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

**Example:**

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

**Examples:**

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

**Example:**

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

**Example:**

```json
{
  "id": "/agent 0",
  "title": "More options",
  "description": "For more topic options"
}
```

## Webhooks

To manage a case or reply to a message, chatbot can send a request to the dedicated webhook URL provided by OneTalk.
