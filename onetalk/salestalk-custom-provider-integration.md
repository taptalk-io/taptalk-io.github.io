# SalesTalk - Custom Provider Integration

* [Overview](#overview)
* [API Specifications](#api-specifications)
* [References](#references)
  - [Get Profile](#get-profile)
  - [Get Category List](#get-category-list)
  - [Get Product List](#get-product-list)
  - [Get Product Details](#get-product-details)
* [Object Models](#object-models)
  - [Category](#category)
  - [Product](#product)
  - [ProductPrice](#productprice)


## Overview

This document describes the requirements for development of custom provider for SalesTalk.

With custom integration, business must provide an API URL which SalesTalk will use to communicate with the custom provider via API. The API must be able to serve different response based on a request's parameter.


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
| action | string | The requested action (e.g.: "get_profile", "get_product_list", etc.)
| data   | object | The request parameters based on the request type.

Example:

```json
{
  "action": "get_profile|get_category_list|get_product_list|get_product_details",
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

### Get Profile

Get the store's profile. It may also be used to check if SalesTalk can connect to the custom provider's API.

**Request Data**

N/A

**Request Example**

```json
{
  "action": "get_profile",
  "data": {}
}
```

**Response Data**

| Field      | Type   | Description
|:-----------|:-------|:----------------------------------------------
| id         | string | The profile ID.
| name       | string | The profile's name or store name.
| pictureURL | string | `Optional` The profile's picture URL, if any.

**Response Example**

```json
{
  "id": "store-001",
  "name": "My Store",
  "pictureURL": "https://www.example.com/store.jpg"
}
```


### Get Category List

Get the list of available categories.

The category ID may contain any value as long as it can be handled by the API when product list is requested with the category ID value.

**Request Data**

N/A

**Request Example**

```json
{
  "action": "get_category_list",
  "data": {}
}
```

**Response Data**

| Field      | Type                    | Description
|:-----------|:------------------------|:------------------------
| items      | [Category](#category)[] | The list of categories.

**Response Example**

```json
{
  "items": [
    {
      "id": "all",
      "name": "All"
    },
    {
      "id": "curated:best-seller",
      "name": "Best Seller"
    },
    {
      "id": "category-001",
      "name": "Accessories"
    }
  ]
}
```

In the example above, "All" and "Best Seller" are not actual categories while "Accessories" is an actual category.
In this way, the API can return any kind of categories as long as it can handle the value in [Get Product List](#get-product-list).


### Get Product List

Get the list of products with pagination.

**Request Data**

| Field      | Type    | Description
|:-----------|:--------|:---------------------------------------------------------
| search     | string  | `Optional` The search term.
| categoryID | string  | The category ID.
| pageNumber | integer | The page number.
| pageSize   | integer | Number of items per page.
| sortBy     | string  | How to sort the list. See possible sorting fields below.
| sortOrder  | string  | Sorting order ("ASC" or "DESC").

Possible sorting fields:
- "id"
- "name"
- "category"
- "unitPrice"

**Request Example**

```json
{
  "action": "get_product_list",
  "data": {
    "search": "",
    "categoryID": "curated:best-seller",
    "pageNumber": 1,
    "pageSize": 20,
    "sortBy": "name",
    "sortOrder": "ASC"
  }
}
```

**Response Data**

| Field      | Type                    | Description
|:-----------|:------------------------|:---------------------------------------------------
| items      | [Product](#product)[]   | The list of products.
| hasMore    | boolean                 | If there are more items in pagination.
| totalItems | integer                 | Total number of items in pagination.
| totalPages | integer                 | Total number of pages for the requested page size.

> Make sure to calculate the `totalItems` and `totalPages` correctly.

**Response Example**

```json
{
  "items": [
    {
      "id": "123",
      "name": "Apple iPhone 14 Pro Max 128GB",
      "category": {
        "id": "category-002",
        "name": "iPhone"
      },
      "imageURL": "https://www.example.com/product-image/123.jpg",
      "description": "Screen size: 6.7\", 1290 x 2796 pixels\nMemory: RAM 6 GB, ROM 128 GB...",
      "prices": [
        {
          "currency": "IDR",
          "unitPrice": 21999000
        }
      ],
      "stock": 12,
      "productURL": "https://www.example.com/product/apple-iphone-14-pro-max-128gb"
    },
    {
      "id": "456",
      "name": "Charger 24W QC3.0",
      "category": {
        "id": "category-001",
        "name": "Accessories"
      },
      "imageURL": "https://www.example.com/product-image/456.jpg",
      "description": "Input: 100-240V - 50/60Hz 0.8A\nOutput: 3.6V...",
      "prices": [
        {
          "currency": "IDR",
          "unitPrice": 89000
        }
      ],
      "stock": 99,
      "productURL": "https://www.example.com/product/charger-24w-qc-30"
    },
    ...
  ],
  "hasMore": false,
  "totalItems": 5,
  "totalPages": 1
}
```


### Get Product Details

Get the details of a product.

**Request Data**

| Field      | Type    | Description
|:-----------|:--------|:----------------
| id         | string  | The product ID.

**Request Example**

```json
{
  "action": "get_product_details",
  "data": {
    "id": "123"
  }
}
```

**Response Data**

| Field      | Type                  | Description
|:-----------|:----------------------|:-----------------------
| item       | [Product](#product)   | The product's details.

**Response Example**

```json
{
  "item": {
    "id": "123",
    "name": "Apple iPhone 14 Pro Max 128GB",
    "category": {
      "id": "category-002",
      "name": "iPhone"
    },
    "imageURL": "https://www.example.com/product-image/123.jpg",
    "description": "Screen size: 6.7\", 1290 x 2796 pixels\nMemory: RAM 6 GB, ROM 128 GB...",
    "prices": [
      {
        "currency": "IDR",
        "unitPrice": 21999000
      }
    ],
    "stock": 12,
    "productURL": "https://www.example.com/product/apple-iphone-14-pro-max-128gb"
  }
}
```



## Object Models

### Category

| Field      | Type   | Description
|:-----------|:-------|:-------------------
| id         | string | The category ID.
| name       | string | The category name.

**Example:**

```json
{
  "id": "category-001",
  "name": "Accessories"
}
```


### Product

| Field       | Type                            | Description
|:------------|:--------------------------------|:------------------------------------------------------
| id          | string                          | The product ID.
| name        | string                          | The product name.
| category    | [Category](#category)           | The product's category.
| imageURL    | string                          | The product's image URL.
| description | string                          | The product's description.
| prices      | [ProductPrice](#productprice)[] | The product's prices (must contain at least 1 price).
| stock       | integer                         | The product's stock.
| productURL  | string                          | The product URL.

**Example:**

```json
{
  "id": "123",
  "name": "Apple iPhone 14 Pro Max 128GB",
  "category": {
    "id": "category-002",
    "name": "iPhone"
  },
  "imageURL": "https://www.example.com/product-image/123.jpg",
  "description": "Screen size: 6.7\", 1290 x 2796 pixels\nMemory: RAM 6 GB, ROM 128 GB...",
  "prices": [
    {
      "currency": "IDR",
      "unitPrice": 21999000
    }
  ],
  "stock": 12,
  "productURL": "https://www.example.com/product/apple-iphone-14-pro-max-128gb"
}
```


### ProductPrice

| Field      | Type    | Description
|:-----------|:--------|:----------------
| currency   | string  | The currency.
| unitPrice  | integer | The unit price.

**Example:**

```json
{
  "currency": "IDR",
  "unitPrice": 89000
}
```
