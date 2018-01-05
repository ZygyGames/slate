---
title: API Reference

language_tabs:
  - shell

toc_footers:

includes:

search: true
---

<!--
Hi Rocco! Do this:

bundle install
bundle exec middleman server
*Make changes, commit*
And then: ./deploy.rb
-->

# Introduction

Welcome to the Activity Maximizer API Documentation. This will include basic information for each endpoint as well as possible responses.

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
  -d "search=What+is+this"

# Multiple parameters in single request separated by `&`. No spaces are used in parameters.
# Spaces, where necessary, are replaced with `+`
curl -X GET www.google.com
  -d "username=myusername&password=my+super+secure+password"

curl -X GET www.google.com
  --data-urlencode "username=myusername"
  --data-urlencode "password=my super secure password"
  # Notice the spaces in the password
```

`-d` contains the params. Unlike the Headers, params are all shown together in URL Parameter format. Normally, params are attached to the url by appending the parameterized string onto the end of the url request with a `?`

You can also use `--data-urlencode` to include params, but only 1 param can be added at a time. However, you do not need to escape the params with plus codes or percent codes.

# Authentication

In order to authenticate with endpoints that require a signed in user, you must pass the authentication token that belongs to the user. You must "login" to retrieve that token using an email/password combination.

Any time that the server responds with a `401` status code, the user should be logged out of the current session and asked to sign in again. The only time this should happen is if they change their password.

## Login

This endpoint authorizes a user.

### HTTP Request

`GET https://maxactivity.com/api/v1/login`

> Example request

```shell
curl -X GET "https://maxactivity.com/api/v1/login"
  -d "email=xxxxx&password=xxxxx"
```

### Request Expectations:

Type | Parameter | Required? | Description
---- | --------- | --------- | -----------
param | `email` | YES | Email Address
param | `password` | sometimes | Password supplied by user

> Response - Failure

```shell
status 401 - Unauthorized
{
  "errors": ["Invalid email or password."]
}
```

> Response - No Password

```shell
status 406 - Not Acceptable
{
  "errors": ["User must set password - Server will deliver email to entered email address."]
}
```

> Response - Success

```shell
-H Authorization Token: asqmbvwwzymwccmbifgvqtfin3t

{  
  "user":{  
    "id": 4336,
    "upline_id": 4,
    "trainer_id": 5,
    "downline_ids": [  
      1, 2, 3
    ],
    "email": "rocco@zygy.com",
    "family_name": "Nicholls",
    "given_name": "Rocco",
    "phone_number": "3852599640",
    "profile_picture_url": "",
    "state": "",
    "solution_number": "",
    "is_rvp": false,
    "firebase_id": "",
    "firebase_ref": "",
    "created_at": "2018-01-05T00:02:50.121Z",
    "updated_at": "2018-01-05T02:07:49.512Z",
    "current_speed": 0
  }
}
```

### Response Expectations

Type | Key | Success? | Description
---- | --- | -------- | -----------
json | `errors` | NO | An array of errors that caused the request to fail
header | `Authorization Token` | YES | The authorization token for the current user. This is used to authorize the user.
json | `id` | YES | Server id for the user.
json | `upline_id` | YES | Server id of the user's upline - MAY BE `null`
json | `trainer_id` | YES | Server id of the user's trainer - MAY BE `null`
json | `downline_ids` | YES | An array of all ids of each of the user's baseshop (Recurring downlines)
json | `email` | YES |
json | `given_name` | YES |
json | `family_name` | YES |
json | `phone_number` | YES |
json | `profile_picture_url` | YES |
json | `state` | YES |
json | `solution_number` | YES |
json | `is_rvp` | YES |
json | `firebase_id` | YES |
json | `firebase_ref` | YES |
json | `created_at` | YES |
json | `updated_at` | YES |
json | `current_speed` | YES | Calculated current speed of the user

<aside class="success">
Remember — Store the `Authorization Token` in-app and over-write it on every request! If the user changes their password, that token is regenerated. Not saving this token will cause your user to be unauthenticated.
</aside>
