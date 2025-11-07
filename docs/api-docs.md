# Spring Boot REST API

# API Endpoints

# Users

## Endpoint: [HTTP VERB] `users/{user-id}`

**Description**:  
Retrieves a single user

### Method

`GET` | `POST` | `PUT` | `DELETE` (choose one)

### Path Parameters

| Name         | Type    | Required | Description               |
|--------------|---------|----------|---------------------------|
| paramName    | string  | yes      | Description of parameter. |

### Query Parameters

| Name         | Type    | Required | Description               |
|--------------|---------|----------|---------------------------|
| queryParam   | int     | no       | Description of parameter. |

### Request Body

- **Content-Type**: `application/json`
- Example:

```
{ “field1”: “value”, “field2”: “value” }
```
