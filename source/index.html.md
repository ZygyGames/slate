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

## Basic details about requests

> Successful Response Status Codes

```shell
status 200 - OK
# This status states that everything went according to plan and things are good.
# This is the most common status.

status 201 - Created
# This status states that an object was successfully created.
```

> Failure Response Status Codes

```shell

status 400 - Bad Request
{
  "errors": ["param is missing or the value is empty: user"]
}
# This error states that something went wrong with parsing the params that were passed in. Generally, it's because the params were not nested under the resource name that they are updating.

status - 401 Unauthorized
{
  "errors": ["Invalid email or password."]
}
# This error can occur whenever a user enters a bad email/password upon logging in, or when a password changes causes their authorization token to become invalid.
# At that point, a user should be logged out immediately.

status - 403 Forbidden
{
  "errors": ["You are not authorized to do that."]
}
# This error occurs if you attempt to make a request that the current user does not have access to- for example, attempting to view somebody not on their team.

status - 406 Not Acceptable
{
  "errors": ["User must set password - Server will deliver email to entered email address."]
}
# This is a special error alerting the app that this user has not set up their password on the migrated server yet. It should be handled the same as a 401, either logging the user out or displaying an invalid email/password error.

status 500 - Server Error
{
  "errors": ["Something went wrong."]
}
# If a 500 is returned, it means something went wrong on the server side. This should be reported, when possible as it generally means there is a crash.

status 502 - Bad Gateway (Could not connect to server)
# (No response)
# This error generally occurs while the server is restarting, but informs the app that the server could not be reached for some reason or another.
```

The basic response codes are shown on the right. These status codes are the HTTP statuses that are made.
Generally, when interacting with maxactivity.com, any time something goes wrong, you will receive a response in the form of a hash/dictionary/associative array with a single key, `errors`. The value for that key is an array of human-readable messages. **These messages are NOT for display to users, but to be used for debugging.** Each endpoint should have custom error messages written in the app to display to the user.

* 1xx: **Information** (Unused)
* 2xx: **Success** - Everything went according to plan
* 3xx: **Redirect** - The url you hit is no longer officially supported and you are being automatically redirected to a different url.
* 4xx: **Client Error** - Something was wrong with the request- generally this means you forgot a required parameter, but could also be an invalid url, requesting an object you don't have permission to view, or otherwise doing something outside of your permissions.
* 5xx: **Server Error** - Something went wrong with the server. This could be a bug in the code or that the server is down. Generally these mean something is going REALLY wrong.

You can read more about status codes [Here](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes)

### Pagination

All requests that return more than a single resource will be paginated. Pagination means the results are given in "pages", to prevent a massive request being sent when only a few results are desired.

**Request**

There are 3 key parameters in pagination. `page`, `per`, and `all`

Defaults:

Param | Default | Description
---- | --------- | ---------
page | `1` | Which "page" of objects to return.
per | `25` | How many objects to return per page.
all | `false` | When set to `true`, pagination is not applied and all results are returned. (Not recommended unless you know the request does not have a lot of objects.)

For example, when `page: 2, per: 50` is given in the parameters, you will receive objects in the list between indices `51-100` (Page 1 would be `1-50` since `per` is set to return 50 objects at a time)

**Response**

Any time you make a request to an endpoint that returns paginated objects, the `meta` data in your response will contain details about the paginated objects.

```shell
Example response with pagination meta details
{
  "users": [{...}, {...}, {...}],
  "meta": {
    "total_count": 512,
    "total_pages": 11,
    "page": 2, # Current page
    "next_page": 3, # Null on the last page
    "prev_page": 1, # Null on the first page
    "per": 50,
  }
}
```

An example response might look like this:

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

The `-X` argument specifies the HTTP Verb. (`GET`, `PUT`, `POST`, `DELETE`, etc...)

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

Errors will always be return in the format of an array, although most of the time there will only be 1 error shown.

## Login

This endpoint authorizes a user.

<aside class="notice">
Note: We should use a generic message for both failure cases. This is to prevent a malicious user entering dozens of emails and figuring out which emails have accounts.

The message should cover both the unknown email/password combo as well as a short message describing the password change.
"Invalid email or password. If you have not migrated your account, you will receive an email shortly with instructions on changing your password."
</aside>

### HTTP Request

`POST https://maxactivity.com/api/v1/login`

> Example request

```shell
curl -X POST "https://maxactivity.com/api/v1/login"
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
status 200 - OK
-H Authorization Token: aaabbbcccdddeeefffggghhhiii

{  
  "user":{  
    "id": 4336,
    ...
  }
}
```

### Response Expectations

Type | Key | Success? | Description
---- | --- | -------- | -----------
json | `errors` | NO | An array of errors that caused the request to fail
header | `Authorization Token` | YES | The authorization token for the current user. This is used to authorize the user.
json | `user` | YES | Data hash of data from the user.

<aside class="warning">
Remember — Store the `Authorization Token` in-app and over-write it on every request! If the user changes their password, that token is regenerated. Not saving this token will cause your user to be unauthenticated.
</aside>

## Password Reset

This endpoint is used to trigger a password reset email for users who have forgotten their password

You can submit an email address OR a solution number. If found in the database, we will deliver a password reset to the email we have on file for that particular user.

This endpoint will ALWAYS return a 200, regardless of if the email is actually sent or the email actually exists within our database. This is a common principle to prevent malicious users from sending batches of emails and figuring out which emails we have in our database.

### HTTP Request

`POST https://maxactivity.com/api/v1/password_reset`

> Example request

```shell
curl -X POST "https://maxactivity.com/api/v1/password_reset"
  -d "email=xxxxx"
```

### Request Expectations:

Type | Parameter | Required? | Description
---- | --------- | --------- | -----------
param | `email` | NO | Email Address
param | `solution_number` | NO | Solution Number

> Response - Success

```shell
status 200 - OK
```

# Users

Use this endpoint to return the attributes of users. This includes the ids of their related associations.

## Show

Returns the data of a single user.

### HTTP Request

`GET https://maxactivity.com/api/v1/users/:id`

This user must be accessible from the current user, meaning they are either an Upline, Trainer, or a Downline of the current user.

> Example request

```shell
curl -X GET "https://maxactivity.com/api/v1/users/4336"
  -H "Authorization: Token aaabbbcccdddeeefffggghhhiii"
```

### Request Expectations:

Type | Parameter | Required? | Description
---- | --------- | --------- | -----------
header | `Authorization` | YES | User authentication value retrieved from login
param | `id` | YES | Server ID of the user to be returned (Included in the URL, does not need to be included as a separate parameter)

> Response - Success

```shell
status 200 - OK
-H Authorization Token: aaabbbcccdddeeefffggghhhiii

{  
  "user":{  
    "id":4336,
    "upline_id":2,
    "trainer_id":3,
    "downline_ids":[  
      4, 5, 6
    ],
    "email":"rocco@zygy.com",
    "family_name":"Nicholls",
    "given_name":"Rocco",
    "phone_number":"3852599640",
    "profile_picture_url":"",
    "state":"UT - Utah",
    "solution_number":"xupr7",
    "is_rvp":false,
    "firebase_id":"00XzzxIXXKbwpyBT81KT9qvPKxZ2",
    "firebase_ref":"https://activitymaximizer.firebaseio.com/users/00XzzxIXXKbwpyBT81KT9qvPKxZ2",
    "created_at":"2018-01-05 12:02:50 AM",
    "updated_at":"2018-01-06 07:30:42 PM"
    "current_speed":0
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
json | `created_at` | YES | Timestamp in the format `%Y-%m-%d %I:%M:%S %p`
json | `updated_at` | YES | Timestamp in the format `%Y-%m-%d %I:%M:%S %p`
json | `current_speed` | YES | Calculated current speed of the user

## Index

Returns an array of every associated user to the current user.

### HTTP Request

`GET https://maxactivity.com/api/v1/users`

Only users who the current user is able to access are returned:
Upline, Trainer, and all downlines are available.

This endpoint is paginated, meaning only a maximum of `per` users are returned.
By passing in `page`, you can select which group of uses are returned.

> Example request

```shell
curl -X GET "https://maxactivity.com/api/v1/users"
  -H "Authorization: Token aaabbbcccdddeeefffggghhhiii"
```

### Request Expectations:

Type | Parameter | Required? | Description
---- | --------- | --------- | -----------
header | `Authorization` | YES | User authentication value retrieved from login
param | `user_ids` | NO | Array of specific user ids to request.
param | `page` | NO | Default: 1 - Page of users to return based on `per`
param | `per` | NO | Default: 25 - Number of users to return per `page`
param | `all` | NO | Default: false - When passed as `true`, will not paginate users and will return all of them regardless of the number.

> Response - Success

```shell
status 200 - OK
-H Authorization Token: aaabbbcccdddeeefffggghhhiii

{  
  "users":[  
    {  
      "id":4335,
      ...
      "current_speed":0
    },
    {  
      "id":2,
      ...
      "current_speed":0
    },
    {  
      "id":4,
      ...
      "current_speed":0
    }
  ]
}
```

### Response Expectations

Type | Key | Success? | Description
---- | --- | -------- | -----------
json | `errors` | NO | An array of errors that caused the request to fail
header | `Authorization Token` | YES | The authorization token for the current user. This is used to authorize the user.
json | `users` | YES | Array of requested users, each object contains the full data returned by the `show` endpoint

## Create

Creates a new user

### HTTP Request

`POST https://maxactivity.com/api/v1/users`

This endpoint does NOT require authentication.

> Example request

```shell
curl -X POST "https://maxactivity.com/api/v1/users"
  -d "user[email]=fredericksmith@email.com"
  -d "user[password]=password"
  -d "user[given_name]=Frederick"

# This is the equivalent of:
{
  "user": {
    "email": "fredericksmith@email.com",
    "password": "password",
    "given_name": "Frederick"
  }
}
```

### Request Expectations:

Type | Parameter | Required? | Description
---- | --------- | --------- | -----------
param | `user` | YES | JSON object containing all of the attributes for the new user.

> Response - Success

```shell
status 201 - Created
-H Authorization Token: aaabbbcccdddeeefffggghhhiii

{  
  "user":{  
    "id": 4336,
    "email": "fredericksmith@email.com",
    "given_name": "Frederick"
    ...
  }
}
```

### Response Expectations

Type | Key | Success? | Description
---- | --- | -------- | -----------
json | `errors` | NO | An array of errors that caused the request to fail - If the user failed to update, this will explain why.
header | `Authorization Token` | YES | The authorization token for the current user. This is used to authorize the user.
json | `user` | YES | Newly created user object.

## Update

Updates a user

### HTTP Request

`PATCH/PUT https://maxactivity.com/api/v1/users/:id`

This endpoint only allows for updating the current user.

> Example request

```shell
curl -X PATCH "https://maxactivity.com/api/v1/users/4336"
  -H "Authorization: Token aaabbbcccdddeeefffggghhhiii"
  -d "user[given_name]=Frederick"

# This is the equivalent of:
{
  "user": {
    "given_name": "Frederick"
  }
}
```

### Request Expectations:

Type | Parameter | Required? | Description
---- | --------- | --------- | -----------
header | `Authorization` | YES | User authentication value retrieved from login
param | `id` | YES | Server ID of the user to be updated (Included in the URL, does not need to be included as a separate parameter)
param | `user` | YES | JSON object containing the update to change for the user.

> Response - Success

```shell
status 200 - OK
-H Authorization Token: aaabbbcccdddeeefffggghhhiii

{  
  "user":{  
    "id": 4336,
    "given_name": "Frederick"
    ...
  }
}
```

### Response Expectations

Type | Key | Success? | Description
---- | --- | -------- | -----------
json | `errors` | NO | An array of errors that caused the request to fail - If the user failed to update, this will explain why.
header | `Authorization Token` | YES | The authorization token for the current user. This is used to authorize the user.
json | `user` | YES | Updated user object.

## Destroy

Removes a user

### HTTP Request

`DELETE https://maxactivity.com/api/v1/users/:id`

This endpoint only allows for deleting the current user.

> Example request

```shell
curl -X DELETE "https://maxactivity.com/api/v1/users/4336"
  -H "Authorization: Token aaabbbcccdddeeefffggghhhiii"
```

### Request Expectations:

Type | Parameter | Required? | Description
---- | --------- | --------- | -----------
header | `Authorization` | YES | User authentication value retrieved from login
param | `id` | YES | Server ID of the user to be deleted (Included in the URL, does not need to be included as a separate parameter)

> Response - Success

```shell
status 200 - OK

# No response is returned after deleting an object.
```

### Response Expectations

Type | Key | Success? | Description
---- | --- | -------- | -----------
json | `errors` | NO | An array of errors that caused the request to fail - If the user failed to update, this will explain why.
