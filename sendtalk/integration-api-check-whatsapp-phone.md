# SendTalk API Documentation

**Base URL**
```
https://sendtalk-api.taptalk.io/api
```

To integrate with SendTalk API, you must provide an API key in the HTTP request header.
To create an API key, go to **SendTalk Dashboard > API > API Keys > Create API Key**.

**HTTP Headers**

| Header  | Description
|:--------|:-------------------------------------------
| API-Key | Integration API key for accessing the API.

**Common Response Payload**

For a valid API request, the response payload will have the following JSON structure:

```json
HTTP/1.1 200 OK
{
  "status": 200,
  "error": {
    "code": "",
    "message": "",
    "field": ""
  },
  "data": {
    DATA...
  }
}
```

The field `data` will contain response body for each different API.

For an invalid API request, the response payload will have the following JSON structure:

```json
HTTP/1.1 200 OK
{
  "status": 400,
  "error": {
    "code": "40002",
    "message": "Phone is required",
    "field": "phone"
  },
  "data": {}
}
```

## SendTalk API - Check WhatsApp Phone

Check if a phone number is registered in WhatsApp.

**Endpoint**

```
/v1/message/check_whatsapp_phone
```

**HTTP Method**

```
POST
```

**Request Body**

| Field | Type   | Description
|:------|:-------|:---------------------------------------------------------
| phone | string | The phone number to check, must start with country code.

Example:

```json
{
    "phone": "6281212345678"
}
```

**Response Body**

| Field        | Type    | Description
|:-------------|:--------|:-----------------------------------------------
| isRegistered | boolean | If the phone number is registered in WhatsApp.
| message      | string  | The message.

The response data above will always be put inside `data` for a successful request.

Example:

```json
HTTP/1.1 200 OK
{
  "status": 200,
  "error": {
    "code": "",
    "message": "",
    "field": ""
  },
  "data": {
    "isRegistered": true,
    "message": "Phone number '6281212345678' is registered"
  }
}
```

```json
HTTP/1.1 200 OK
{
  "status": 200,
  "error": {
    "code": "",
    "message": "",
    "field": ""
  },
  "data": {
    "isRegistered": false,
    "message": "Phone number '6281212345678' is not registered"
  }
}
```
