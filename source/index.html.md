---
title: API Reference

language_tabs:
  - shell

toc_footers:
  - <a href='https://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:

search: true
---

# Introduction

Welcome to the Zygy Games API Documentation. This will include basic information for each endpoint as well as possible responses.

All of the examples will be given in `curl` (for Requests TO the server) and `JSON` (for responses FROM the server)

This is to retain a consistent syntax, since `curl` is easy to read and understand.

# Curl

## What is `curl`?

```shell
curl www.google.com
```

`curl` is a Command Line tool used for data transfer with URL syntax.

A basic `curl` request contains any url.

It accepts arguments by specifying an argument type followed by the argument.

```shell
curl -X GET www.google.com
```

For this documentation, we will only be using 3 different arguments.

`-X`, `-H`, and `-d`

The arguments are not order-specific, as long as the argument value immediately follows it's flag.

```shell
curl -X GET www.google.com
-H "MyHeader: ThisIsMyHeaderData"
-H "MyOtherHeader: ThisIsMyOtherHeaderData"
```

The `-X` argument specifies the HTML Verb. (`GET`, `PUT`, `POST`, `DELETE`, etc...)

Normally arguments are all done in-line, but for this documentation, we will split keep the `-X` inline and drop the headers and data onto following lines for readability.

`-H` contains the Headers. Headers are shown 1 at a time in `key: value` format. All params/headers/urls are assumed to be strings, so no quotes are necessary.

```shell
# Request by parameter appending
curl -X GET www.google.com?search=What+is+this

# Request by using data/param flag
curl -X GET www.google.com
-d "search=What+is+this'

# Multiple parameters in single request separated by `&`. No spaces are used in parameters.
# Spaces, where necessary, are replaced with `+`
curl -X GET www.google.com
-d "username=myusername&password=my+super+secure+password'
```

`-d` contains the params. Unlike the Headers, params are all shown together in URL Parameter format. Normally, params are attached to the url by appending the parameterized string onto the end of the url request with a `?`

# Authentication

Every request should include the App-Identifier. All non-authorization requests should include the App-Identifier and the Authorization-Code retrieved from Authorization.

Every request will respond with a `Authorization-Success` header with a value of a stringified boolean.

The `Authorization-Code` contains a 20 character alpha-numeric string that should be stored on the app. Every non-authorization request will require this code in order to authorize the user.

Every time a user logs in to the Zygy App, this code will be regenerated. The code will persist across sessions, but the user can only be logged in to 1 device at a time, so logging in to a different device will cause the old code to expire.

In the case of an Authorization Failure because of an expired `Authorization-Code`, the user should be logged out of the app and taken back to the Login screen.

<aside class="notice">
Not sure what to use for the App-Identifier? Contact Rocco. For security reasons, the App-Identifier's will not be posted in the docs.
</aside>

## Login

This endpoint authorizes a user.

### HTTP Request

`GET https://zygygames.com/zygy_app_api/login`

### Request expectations:

```shell
curl -X GET "https://zygygames.com/zygy_app_api/login"
-H "App-Identifier: 123456789-987654321"
-d "identifier_field=xxxxx&password_field=xxxxx"
```

Type | Parameter | Required? | Description
---- | --------- | --------- | -----------
header | `App-Identifier` | true | App Identification String
param | `identifier_field` | true | Username, Email Address, or Zygy ID supplied by user
param | `password_field` | true | Password supplied by user

### Response Expectations

> Response - Failure

```shell
-H Authorization-Success : false
-H Error-Message : Invalid Username or Password
```
```
{
  "error": "Invalid Username or Password"
}
```

> Response - Success

```shell
-H Authorization-Success : true
-H Authorization-Code : OUKdeYQf1qiEoTF8clnk
```
```
{
  "username": "JohnDoe67",
  "server_id": "4",
  "zygy_id": "XUPR71"
}
```

Type | Key | Success? | Description
---- | --- | -------- | -----------
header | `Authorization-Success` | N/A | Stringified boolean representing whether or not the request was successful.
header | `Error-Message` | false | Same message as json-error, passed as a Header.
json | `error` | false | Description of why a failure resulted - Displayable to user
header | `Authorization-Code` | true | Authorization Code for user. Save this within the app.
json | `username` | true | Username of logged in user.
json | `server_id` | true | Id for user on the server for URL requests only. (Uncommonly used)
json | `zygy_id` | true |  6 digit Zygy ID of user. Commonly used and displayable

<aside class="success">
Remember — Store the `Authorization-Code` in-app and over-write it every time a user logs in!
</aside>

## Register

This endpoint authorizes a new user. All fields are required.

### HTTP Request

`GET https://zygygames.com/zygy_app_api/register`

### Request expectations:

```shell
curl -X GET "https://zygygames.com/zygy_app_api/register"
-H "App-Identifier: 123456789-987654321"
-d "username=xxxxx&referral_code=xxxxxx&first_name=xxxxx&last_name=xxxxx&email=xxxxx@xxx.com&confirm_email=xxxxx@xxx.com&birthday=xx/xx/xx&password=xxxxxxxx&confirm_password=xxxxxxxx"
```


Type | Parameter | Required? | Description
---- | --------- | --------- | -----------
header | `App-Identifier` | true | App Identification String
param | `username` | true | Desired username. Must be between 2 and 20 characters
param | `referral_code` | true | Zygy ID of the users friend that referred them to Zygy.
param | `first_name` | true | User's First Name
param | `last_name` | true | User's Last Name
param | `email` | true | User's email address
param | `confirm_email` | true | Confirmation email, must match `email` field.
param | `birthday` | true | Birthday in the format MM/DD/YYYY
param | `password` | true | Desired password. Must be 8 characters or more.
param | `confirm_password` | true | Confirm password - Must match `password` field

### Response Expectations

> Response - Failure

```shell
-H Authorization-Success : false
-H Error-Message : Username already taken.
```
```
{
  "error": "Username already taken."
}
```

> Response - Success

```shell
-H Authorization-Success : true
-H Authorization-Code : OUKdeYQf1qiEoTF8clnk
```
```
{
  "username": "JohnDoe67",
  "server_id": "4",
  "zygy_id": "XUPR71"
}
```

Type | Key | Success? | Description
---- | --- | -------- | -----------
header | `Authorization-Success` | N/A | Stringified boolean representing whether or not the request was successful.
header | `Error-Message` | false | Same message as json-error, passed as a Header.
json | `error` | false | Description of why a failure resulted - Displayable to user
header | `Authorization-Code` | true | Authorization Code for user. Save this within the app.
json | `username` | true | Username of logged in user.
json | `server_id` | true | Id for user on the server for URL requests only. (Uncommonly used)
json | `zygy_id` | true |  6 digit Zygy ID of user. Commonly used and displayable

<aside class="success">
Remember — Store the `Authorization-Code` in-app and over-write it every time a user logs in!
</aside>
