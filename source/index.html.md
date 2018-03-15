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
http://localhost:4567
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
Generally, when interacting with maxactivity.com, any time something goes wrong, you will receive a response in the form of a key/value/dictionary/associative array with a single key, `errors`. The value for that key is an array of human-readable messages. **These messages are NOT for display to users, but to be used for debugging.** Each endpoint should have custom error messages written in the app to display to the user.

* 1xx: **Information** (Unused)
* 2xx: **Success** - Everything went according to plan
* 3xx: **Redirect** - The url you hit is no longer officially supported and you are being automatically redirected to a different url.
* 4xx: **Client Error** - Something was wrong with the request- generally this means you forgot a required parameter, but could also be an invalid url, requesting an object you don't have permission to view, or otherwise doing something outside of your permissions.
* 5xx: **Server Error** - Something went wrong with the server. This could be a bug in the code or that the server is down. Generally these mean something is going REALLY wrong.

You can read more about status codes [Here](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes)

### Formats

`timestamp`s are always returned as strings in UTC with the format:

`YYYY-MM-DD HH:MM:SS P`

Example: `2018-01-09 08:30:47 PM`


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
  "users": [{}, {}, {}],
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
    "..."
  }
}
```

### Response Expectations

Type | Key | Success? | Description
---- | --- | -------- | -----------
json | `errors` | NO | `key/value:array<string>`
header | `Authorization Token` | YES | `string`
json | `user` | YES | `key/value:userJson`

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
param | `email` | NO | `string`
param | `solution_number` | NO | `string`

> Response - Success

```shell
status 200 - OK
```

# Updates / Websockets

## Updates

This endpoint requires a `last_sync` parameter to be sent, and will respond with all viewable objects that have been updated or deleted since the requested timestamp.

> Example request

```shell
curl -X GET "https://maxactivity.com/api/v1/updates" \
  -H "Authorization: Token aaabbbcccdddeeefffggghhhiii" \
  --data-urlencode "last_sync=2018-02-01 04:00:00 PM"
```

> Response

```shell
{  
  "timestamp": "2018-02-01 04:00:00 PM"
  "changed":[
    "users": [{}, {}, {}],
    "contacts": [{}, {}],
    "lists": [{}, {}]
  ],
  "deleted":[
    "contacts": [{}]
  ]
}
```

`GET https://maxactivity.com/api/v1/updates`

The response from this endpoint is a complex object. At the top level there are 3 keys: `changed` (Array), `deleted` (Array), `timestamp` (Timestamp/String)

The `timestamp` is the current time when the request was made. (This can/should be stored for later)
The `changed` and `deleted` arrays contain the objects that were changed or deleted, respectively.
The objects are key/value pairs where the key is the object type and the value is an array of the objects of that type.

## Websockets

> Example request

```shell
# Connect:
wscat -c "wss://maxactivity.com/cable" -o "https://maxactivity.com" \
  -H "Authorization: Token aaabbbcccdddeeefffggghhhiii"

# Subscribe:
{
  "command": "subscribe",
  "identifier": "{\"channel\":\"UpdatesChannel\",\"channel_id\":\"updates_1111\"}"
}
# Notice that `identifier` is a string value with JSON inside.

# Message:
{
  "command": "message",
  "identifier": "{\"channel\":\"UpdatesChannel\",\"channel_id\":\"updates_1111\"}",
  "data": "{\"action\":\"pong\",\"content\":\"Hello, World\"}"
}
# Again, both `identifier` and `data` are Strings with JSON inside.
```

> Response

```shell
{
  "timestamp": "2018-02-04 12:36:44 AM",
  "changed": {
    "users": [
      {
        "id": 1111,
        "upline_id": 1000,
        "trainer_id": 1001,
        "downline_ids": [],
        "email": "rocco@zygy.com",
        "family_name": "Nicholls",
        "given_name": "Rocco",
        "phone_number": "(385)259-9640",
        "profile_picture_url": "",
        "state": "UT - Utah",
        "solution_number": "xupr7",
        "is_rvp": false,
        "created_at": "2017-03-05 04:52:03 AM",
        "updated_at": "2018-02-04 12:36:44 AM",
        "current_speed": 0
      }
    ]
  }
}
```

Websockets allow you to receive updates as they happen. To demonstrate websockets, the `wscat` command line tool will be used.

`wscat` can be installed with `npm install -g wscat`

### Connect

The initial connection must be made to the url:
`wss://maxactivity.com/cable`
The request origin must be set to the root domain,
`https://maxactivity.com`

In order to use the websockets, you must be authorized. You can authorize the same way as other endpoints, by including the `Authorization` header.

<aside class="warning">
If a connection is lost, the app should continually attempt to reconnect to the server until the connection is once again made. At that point, you must also subscribe again.
</aside>

### Subscribe

In order to actually get data back from the server, you must first subscribe to a channel.

The connection is like having a cell phone network, but subscribing to a channel is actually dialing a phone number.

In order to keep things simple, there is only 1 channel you must subscribe to- the `UpdatesChannel`

In order to subscribe, you must send data through the websocket telling the server which channel you want to subscribe to.

`command` tells the server what you want to do- `subscribe`.

`identifier` is a **String** containing the subscription data. Specifically, in that string, you want to pass the `channel` you are subscribing to and the `channel_id`, which is a string telling the server which user you want updates for.

The `channel` is a hard set string as `UpdatesChannel`
The `channel_id` is a string `updates_1111` - replace `1111` with the current user's server id.

### Response

The response from the server will not come immediately, but as a message through the websocket. The message will always be a JSON packet. This data will exactly match the format of the `updates` endpoint, but typically only 1 object will be sent.  

# Account

## Month End

This endpoint simply returns the timestamp for the next month-end date.
Because this can change at any time, this endpoint should be checked often.

**This Endpoint does not require authentication**

`GET https://maxactivity.com/api/v1/account/month_end`

The response from this endpoint is simply a string containing the formatted date for the month end of the current month.

You can optionally add a `date` parameter to get the month end for a specific date.

<aside class="notice">
The Month End date is calculated as the first week day in the following month, and skips Labor Day and New Years Day.
</aside>

> Example request

```shell
curl -X GET "https://maxactivity.com/api/v1/account/month_end"
```

> Response

```shell
2018-02-01 04:00:00 PM
```

## OneSignal Notifications

Once the user has subscribed to OneSignal Notifications, send their OneSignal ID to this endpoint in order to allow remote notifications from the server.

`POST https://maxactivity.com/api/v1/account/subscribe_notifications`

> Example request

```shell
curl -X GET "https://maxactivity.com/api/v1/account/subscribe_notifications"
  -d "onesignal_id=asdfasdf-asdfasdf-asdfasdf"
```

# Users

Use this endpoint to return the attributes of users.

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
header | `Authorization` | YES | `string`
param | `id` | YES | `integer`

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
    "created_at":"2018-01-05 12:02:50 AM",
    "updated_at":"2018-01-06 07:30:42 PM"
    "current_speed":0
  }
}
```

### Response Expectations

Type | Key | Success? | Description
---- | --- | -------- | -----------
json | `errors` | NO | `key/value:array<string>`
header | `Authorization Token` | YES | `string`
json | `id` | YES | `integer`
json | `upline_id` | YES | `integer` (nullable)
json | `trainer_id` | YES | `integer` (nullable)
json | `downline_ids` | YES | `array<integer>`
json | `email` | YES | `string`
json | `given_name` | YES | `string`
json | `family_name` | YES | `string`
json | `phone_number` | YES | `string`
json | `profile_picture_url` | YES | `string`
json | `state` | YES | `string`
json | `solution_number` | YES | `string`
json | `is_rvp` | YES | `boolean`
json | `created_at` | YES | `timestamp`
json | `updated_at` | YES | `timestamp`
json | `current_speed` | YES | `integer`

## Index

Returns an array of every associated user to the current user.

### HTTP Request

`GET https://maxactivity.com/api/v1/users`

Only users who the current user is able to access are returned:
Upline, Trainer, and all downlines are available.

This endpoint is paginated, meaning only a maximum of `per` users are returned.
By passing in `page`, you can select which group of users are returned.

**Filterable Options**

`last_sync` pass in a timestamp to return only the objects updated LATER than the requested date

`since` pass in a timestamp to return only the objects updated EARLIER than the requested date

`user_ids`, the endpoint will only return the included users.

> Example request

```shell
curl -X GET "https://maxactivity.com/api/v1/users"
  -H "Authorization: Token aaabbbcccdddeeefffggghhhiii"
```

### Request Expectations:

Type | Parameter | Required? | Description
---- | --------- | --------- | -----------
header | `Authorization` | YES | `string`
param | `user_ids` | NO | `array<integer>`
param | `last_sync` | NO | `timestamp`
param | `page` | NO | `integer` (Default: `1`)
param | `per` | NO | `integer` (Default: `25`)
param | `all` | NO | `boolean` (Default: `false`)

> Response - Success

```shell
status 200 - OK
-H Authorization Token: aaabbbcccdddeeefffggghhhiii

{  
  "users":[{}, {}, {}],
  "meta": {}
}
```

### Response Expectations

Type | Key | Success? | Description
---- | --- | -------- | -----------
json | `errors` | NO | `key/value:array<string>`
header | `Authorization Token` | YES | `string`
json | `users` | YES | `array<key/value:userJson>`
json | `meta` | YES | `key/value:meta`

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
json | `user` | YES | `key/value:userJson`

> Response - Success

```shell
status 201 - Created
-H Authorization Token: aaabbbcccdddeeefffggghhhiii

{  
  "user":{  
    "id": 4336,
    "email": "fredericksmith@email.com",
    "given_name": "Frederick",
    "..."
  }
}
```

### Response Expectations

Type | Key | Success? | Description
---- | --- | -------- | -----------
json | `errors` | NO | `key/value:array<string>`
json | `user` | YES | `key/value:userJson`

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
header | `Authorization` | YES | `string`
param | `id` | YES | `integer`
json | `user` | YES | `key/value:userJson`

> Response - Success

```shell
status 200 - OK
-H Authorization Token: aaabbbcccdddeeefffggghhhiii

{  
  "user":{  
    "id": 4336,
    "given_name": "Frederick",
    "..."
  }
}
```

### Response Expectations

Type | Key | Success? | Description
---- | --- | -------- | -----------
json | `errors` | NO | `key/value:array<string>`
header | `Authorization Token` | YES | `string`
json | `user` | YES | `key/value:userJson`

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
header | `Authorization` | YES | `string`
param | `id` | YES | `integer`

> Response - Success

```shell
status 200 - OK

# No response is returned after deleting an object.
```

### Response Expectations

Type | Key | Success? | Description
---- | --- | -------- | -----------
json | `errors` | NO | `key/value:array<string>`

# Company

> Example request

```shell
curl -X GET "https://maxactivity.com/api/v1/company"
  -H "Authorization: Token aaabbbcccdddeeefffggghhhiii"
```

### Request Expectations:

Type | Parameter | Required? | Description
---- | --------- | --------- | -----------
header | `Authorization` | YES | `string`

> Response - Success

```shell
status 200 - OK
-H Authorization Token: aaabbbcccdddeeefffggghhhiii

{  
  "company":{  
    "id":1,
    "name":"None",
    "event_groups":{
      "Appointment":[
        "Set Appointment",
        "Went on Appointment"
      ],
      "Other":[
        "Invited to Meeting",
        "Went to Meeting"
      ]
    },
    "rating_groups":{
      "Client Rating":[
        "Has Kids",
        "Homeowner",
        "Income Over 40k",
        "Married",
        "Age 25-55"
      ],
      "Recruit Rating":[
        "Competitive",
        "Credible",
        "Hungry",
        "Motivated",
        "People Skills"
      ]
    },
    "goal_types":[
      "Contacts Added",
      "Set Appointment",
      "Went on Appointment",
      "Invited to Meeting",
      "Went to Meeting"
    ]
  }
}
```

Use this endpoint to return the company data of the current user

### HTTP Request

`GET https://maxactivity.com/api/v1/company`

Currently, the company has data for event and rating groups.

Event Groups is nested hash which contains the groups of Event Types.

Rating Groups is a nested hash which contains the groups of Rating Types.

# Contacts

Use this endpoint to return the attributes of contacts for the current user.

## Show

Returns the data of a single contact.

### HTTP Request

`GET https://maxactivity.com/api/v1/contacts/:id`

This contact must be accessible from the current user, meaning the contact must belong to them or one of their downlines.

> Example request

```shell
curl -X GET "https://maxactivity.com/api/v1/contacts/4336"
  -H "Authorization: Token aaabbbcccdddeeefffggghhhiii"
```

### Request Expectations:

Type | Parameter | Required? | Description
---- | --------- | --------- | -----------
header | `Authorization` | YES | `string`
param | `id` | YES | `integer`

> Response - Success

```shell
status 200 - OK
-H Authorization Token: aaabbbcccdddeeefffggghhhiii

"contact": {
  "id": 37140,
  "user_id": 1803,
  "family_name": "Nicholls",
  "given_name": "Rocco",
  "phone_number": "(385)259-9640",
  "street1": null,
  "street2": null,
  "city": null,
  "state": null,
  "zip": null,
  "country": null,
  "ratings": {
    "Client Rating": {
      "Has Kids": false,
      "Homeowner": true,
      "Income Over 40k": true,
      "Married": true,
      "Age 25-55": true
    },
    "Recruit Rating": {
      "Competitive": false,
      "Credible": true,
      "Hungry": true,
      "Motivated": true,
      "People Skills": false
    }
  },
  "client_rating": 4,
  "recruit_rating": 3,
  "credibility_rating": 10,
  "imported": true,
  "recruited": false,
  "invited": false,
  "sold": false,
  "attended": false,
  "list_ids": [
    10813,
    10817,
    26704
  ],
  "created_at": "2017-12-22 05:06:36 PM",
  "updated_at": "2018-02-24 01:11:53 AM",
  "notes": [
    {
      "id": 6078,
      "contact_id": 37140,
      "content": "Some Contact Note",
      "created_at": "2018-02-24 01:11:01 AM",
      "updated_at": "2018-02-24 01:11:01 AM"
    }
  ]
}
```

### Response Expectations

Type | Key | Success? | Description
---- | --- | -------- | -----------
json | `errors` | NO | `key/value:array<string>`
header | `Authorization Token` | YES | `string`
json | `id` | YES | `integer`
json | `user_id` | YES | `integer`
json | `family_name` | YES | `string`
json | `given_name` | YES | `string`
json | `email` | YES | `string`
json | `phone_number` | YES | `string`
json | `street` | YES | `string`
json | `street2` | YES | `string`
json | `city` | YES | `string`
json | `state` | YES | `string`
json | `zip` | YES | `string`
json | `country` | YES | `string`
json | `ratings` | YES | `key/value:boolean`
json | `notes` | YES | `array<Note>`
json | `client_rating` | YES | `integer`
json | `recruit_rating` | YES | `integer`
json | `credibility_rating` | YES | `integer`
json | `imported` | YES | `boolean`
json | `created_at` | YES | `timestamp`
json | `updated_at` | YES | `timestamp`

## Index

Returns an array of all contacts viewable by the current user.

### HTTP Request

`GET https://maxactivity.com/api/v1/contacts`

Only contacts who the current user is able to access are returned:
Contacts belonging to the current user or any user lower in the hierarchy.

This endpoint is paginated, meaning only a maximum of `per` contacts are returned.
By passing in `page`, you can select which group of contacts are returned.

**Filterable Options**

`last_sync` pass in a timestamp to return only the objects updated LATER than the requested date

`since` pass in a timestamp to return only the objects updated EARLIER than the requested date

`contact_ids` (Array), only return the contacts requested.

`user_ids` (Array) or `user_id` (Integer), the endpoint will only return contacts belonging to the requested users.

> Example request

```shell
curl -X GET "https://maxactivity.com/api/v1/contacts"
  -H "Authorization: Token aaabbbcccdddeeefffggghhhiii"
```

### Request Expectations:

Type | Parameter | Required? | Description
---- | --------- | --------- | -----------
header | `Authorization` | YES | `string`
param | `user_id` | NO | `integer`
param | `user_ids` | NO | `array<integer>`
param | `last_sync` | NO | `timestamp`
param | `page` | NO | `integer` (Default: `1`)
param | `per` | NO | `integer` (Default: `25`)
param | `all` | NO | `boolean` (Default: `false`)

> Response - Success

```shell
status 200 - OK
-H Authorization Token: aaabbbcccdddeeefffggghhhiii

{  
  "contacts":[{}, {}, {}],
  "meta":{}
}
```

### Response Expectations

Type | Key | Success? | Description
---- | --- | -------- | -----------
json | `errors` | NO | `key/value:array<string>`
header | `Authorization Token` | YES | `string`
json | `contacts` | YES | `array<key/value:contactJson>`
json | `meta` | YES | `key/value:meta`

## Create

Creates a new contact for the current user

### HTTP Request

`POST https://maxactivity.com/api/v1/contacts`

> Example request

```shell
curl -X POST "https://maxactivity.com/api/v1/contacts"
  -H "Authorization: Token aaabbbcccdddeeefffggghhhiii"
  -d "contact[family_name]=Smith"
  -d "contact[given_name]=Sarah"

# This is the equivalent of:
{
  "contact": {
    "given_name": "Sarah",
    "given_name": "Smith"
  }
}
```

### Request Expectations:

Type | Parameter | Required? | Description
---- | --------- | --------- | -----------
header | `Authorization` | YES | `string`
param | `contact` | YES | `key/value:contactJson`

> Response - Success

```shell
status 201 - Created
-H Authorization Token: aaabbbcccdddeeefffggghhhiii

{  
  "contact":{  
    "id": 4336,
    "given_name": "Sarah"
    "family_name": "Smith",
    "..."
  }
}
```

### Response Expectations

Type | Key | Success? | Description
---- | --- | -------- | -----------
json | `errors` | NO | `key/value:array<string>`
header | `Authorization Token` | YES | `string`
json | `contact` | YES | `key/value:contactJson`

## Update

Updates a contact.

### HTTP Request

`PATCH/PUT https://maxactivity.com/api/v1/contacts/:id`

Can only update contacts that belong to the current user
Ratings and Notes can be edited by nesting them

> Example request

```shell
curl -X PATCH "https://maxactivity.com/api/v1/contacts/4336"
  -H "Authorization: Token aaabbbcccdddeeefffggghhhiii"
  -d "contact[given_name]=Frederick"

# This is the equivalent of:
{
  "contact": {
    "given_name": "Frederick"
    "ratings": {
      "Client Rating": {
        "Has Kids": false
      }
    },
    "notes": [
      {"content": "This is adding a new note."},
      {"id": 1, "content": "This is updating an existing note."},
      {"id": 2, "_destroy": true} # This deletes an existing note
    ]
  }
}
```

### Request Expectations:

Type | Parameter | Required? | Description
---- | --------- | --------- | -----------
header | `Authorization` | YES | `string`
param | `id` | YES | `integer`
json | `contact` | YES | `key/value:contactJson`

> Response - Success

```shell
status 200 - OK
-H Authorization Token: aaabbbcccdddeeefffggghhhiii

{  
  "contact":{  
    "id": 4336,
    "given_name": "Frederick",
    "..."
  }
}
```

### Response Expectations

Type | Key | Success? | Description
---- | --- | -------- | -----------
json | `errors` | NO | `key/value:array<string>`
header | `Authorization Token` | YES | `string`
json | `contact` | YES | `key/value:contactJson`

## Destroy

Removes a contact

### HTTP Request

`DELETE https://maxactivity.com/api/v1/contacts/:id`

May only delete contacts that belong to the current user.

> Example request

```shell
curl -X DELETE "https://maxactivity.com/api/v1/contacts/4336"
  -H "Authorization: Token aaabbbcccdddeeefffggghhhiii"
```

### Request Expectations:

Type | Parameter | Required? | Description
---- | --------- | --------- | -----------
header | `Authorization` | YES | `string`
param | `id` | YES | `integer`

> Response - Success

```shell
status 200 - OK

# No response is returned after deleting an object.
```

### Response Expectations

Type | Key | Success? | Description
---- | --- | -------- | -----------
json | `errors` | NO | `key/value:array<string>`

# List Contacts

Use this endpoint to return the join table between Lists and Contacts for backfilling purposes

## Index

Returns the data of a single list.

### HTTP Request

`GET https://maxactivity.com/api/v1/list_contacts`

This list must be accessible from the current user, meaning the list must belong to them or one of their downlines.

> Example request

```shell
curl -X GET "https://maxactivity.com/api/v1/list_contacts"
  -H "Authorization: Token aaabbbcccdddeeefffggghhhiii"
```

### Request Expectations:

Type | Parameter | Required? | Description
---- | --------- | --------- | -----------
header | `Authorization` | YES | `string`
param | `id` | YES | `integer`

> Response - Success

```shell
status 200 - OK
-H Authorization Token: aaabbbcccdddeeefffggghhhiii

{  
  "list_contacts": [
    {
      "list_id": 26706,
      "contact_id": 97278,
      "created_at": "2018-02-10 01:46:49 AM",
      "updated_at": "2018-02-10 01:46:49 AM",
      "deleted_at": null
    }
  ]
}
```

### Response Expectations

Type | Key | Success? | Description
---- | --- | -------- | -----------
json | `errors` | NO | `key/value:array<string>`
header | `Authorization Token` | YES | `string`
json | `id` | YES | `integer`
json | `list_id` | YES | `integer`
json | `contact_id` | YES | `integer`
json | `created_at` | YES | `timestamp`
json | `updated_at` | YES | `timestamp`

# Lists

Use this endpoint to return the attributes of lists for the current user.

## Show

Returns the data of a single list.

### HTTP Request

`GET https://maxactivity.com/api/v1/lists/:id`

This list must be accessible from the current user, meaning the list must belong to them or one of their downlines.

> Example request

```shell
curl -X GET "https://maxactivity.com/api/v1/lists/4336"
  -H "Authorization: Token aaabbbcccdddeeefffggghhhiii"
```

### Request Expectations:

Type | Parameter | Required? | Description
---- | --------- | --------- | -----------
header | `Authorization` | YES | `string`
param | `id` | YES | `integer`

> Response - Success

```shell
status 200 - OK
-H Authorization Token: aaabbbcccdddeeefffggghhhiii

{  
  "list":{  
    "id": 4336,
    "user_id": 1,
    "name": "Cool People",
    "contact_ids": [1, 2, 3],
    "created_at": "2018-01-10 01:03:47 AM",
    "updated_at": "2018-01-10 01:03:47 AM"
  }
}
```

### Response Expectations

Type | Key | Success? | Description
---- | --- | -------- | -----------
json | `errors` | NO | `key/value:array<string>`
header | `Authorization Token` | YES | `string`
json | `id` | YES | `integer`
json | `user_id` | YES | `integer`
json | `name` | YES | `string`
json | `contact_ids` | YES | `array<integer>`
json | `created_at` | YES | `timestamp`
json | `updated_at` | YES | `timestamp`

## Index

Returns an array of all lists viewable by the current user.

### HTTP Request

`GET https://maxactivity.com/api/v1/lists`

Only lists who the current user is able to access are returned:
Lists belonging to the current user or any user lower in the hierarchy.

This endpoint is paginated, meaning only a maximum of `per` lists are returned.
By passing in `page`, you can select which group of lists are returned.

**Filterable Options**

`last_sync` pass in a timestamp to return only the objects updated LATER than the requested date

`since` pass in a timestamp to return only the objects updated EARLIER than the requested date

`contact_ids` (Array) or `contact_id` (Integer), the endpoint will only return lists containing the requested contact(s).

`user_ids` (Array) or `user_id` (Integer), the endpoint will only return lists for the requested user(s).

`list_ids` to only return lists with the passed ids

> Example request

```shell
curl -X GET "https://maxactivity.com/api/v1/lists"
  -H "Authorization: Token aaabbbcccdddeeefffggghhhiii"
```

### Request Expectations:

Type | Parameter | Required? | Description
---- | --------- | --------- | -----------
header | `Authorization` | YES | `string`
param | `user_id` | NO | `integer`
param | `user_ids` | NO | `array<integer>`
param | `last_sync` | NO | `timestamp`
param | `page` | NO | `integer` (Default: `1`)
param | `per` | NO | `integer` (Default: `25`)
param | `all` | NO | `boolean` (Default: `false`)

> Response - Success

```shell
status 200 - OK
-H Authorization Token: aaabbbcccdddeeefffggghhhiii

{  
  "lists":[{}, {}, {}],
  "meta":{}
}
```

### Response Expectations

Type | Key | Success? | Description
---- | --- | -------- | -----------
json | `errors` | NO | `key/value:array<string>`
header | `Authorization Token` | YES | `string`
json | `lists` | YES | `array<key/value:listJson>`
json | `meta` | YES | `key/value:meta`

## Create

Creates a new list for the current user

Optionally, can pass `contact_ids`, `add_contact_ids`, `remove_contact_ids` to add Contacts to the list.

`contact_ids` will overwrite whatever existing Contacts there are.

`add_contact_ids` and `remove_contact_ids` alter the current list of Contacts to include/exclude the passed in Contacts.

### HTTP Request

`POST https://maxactivity.com/api/v1/lists`

> Example request

```shell
curl -X POST "https://maxactivity.com/api/v1/lists"
  -H "Authorization: Token aaabbbcccdddeeefffggghhhiii"
  -d "list[name]=Cool%20People"

# This is the equivalent of:
{
  "list": {
    "name": "Cool People"
  }
}
```

### Request Expectations:

Type | Parameter | Required? | Description
---- | --------- | --------- | -----------
header | `Authorization` | YES | `string`
param | `list` | YES | `key/value:listJson`

> Response - Success

```shell
status 201 - Created
-H Authorization Token: aaabbbcccdddeeefffggghhhiii

{  
  "list":{  
    "id": 4336,
    "name": "Cool People"
    "..."
  }
}
```

### Response Expectations

Type | Key | Success? | Description
---- | --- | -------- | -----------
json | `errors` | NO | `key/value:array<string>`
header | `Authorization Token` | YES | `string`
json | `list` | YES | `key/value:listJson`

## Update

Updates a list.

Optionally, can pass `contact_ids`, `add_contact_ids`, `remove_contact_ids` to add Contacts to the list.

`contact_ids` will overwrite whatever existing Contacts there are.

`add_contact_ids` and `remove_contact_ids` alter the current list of Contacts to include/exclude the passed in Contacts.


### HTTP Request

`PATCH/PUT https://maxactivity.com/api/v1/lists/:id`

Can only update lists that belong to the current user

> Example request

```shell
curl -X PATCH "https://maxactivity.com/api/v1/lists/4336"
  -H "Authorization: Token aaabbbcccdddeeefffggghhhiii"
  -d "list[name]=Cool%20People"

# This is the equivalent of:
{
  "list": {
    "name": "Cool People"
  }
}
```

### Request Expectations:

Type | Parameter | Required? | Description
---- | --------- | --------- | -----------
header | `Authorization` | YES | `string`
param | `id` | YES | `integer`
json | `list` | YES | `key/value:listJson`

> Response - Success

```shell
status 200 - OK
-H Authorization Token: aaabbbcccdddeeefffggghhhiii

{  
  "list":{  
    "id": 4336,
    "name": "Cool People",
    "..."
  }
}
```

### Response Expectations

Type | Key | Success? | Description
---- | --- | -------- | -----------
json | `errors` | NO | `key/value:array<string>`
header | `Authorization Token` | YES | `string`
json | `list` | YES | `key/value:listJson`

## Destroy

Removes a list

### HTTP Request

`DELETE https://maxactivity.com/api/v1/lists/:id`

May only delete lists that belong to the current user.

> Example request

```shell
curl -X DELETE "https://maxactivity.com/api/v1/lists/4336"
  -H "Authorization: Token aaabbbcccdddeeefffggghhhiii"
```

### Request Expectations:

Type | Parameter | Required? | Description
---- | --------- | --------- | -----------
header | `Authorization` | YES | `string`
param | `id` | YES | `integer`

> Response - Success

```shell
status 200 - OK

# No response is returned after deleting an object.
```

### Response Expectations

Type | Key | Success? | Description
---- | --- | -------- | -----------
json | `errors` | NO | `key/value:array<string>`

# Events

Use this endpoint to return the attributes of events for the current user.

## Show

Returns the data of a single event.

### HTTP Request

`GET https://maxactivity.com/api/v1/events/:id`

This event must be accessible from the current user, meaning the event must belong to them,  one of their downlines, or their trainer.

> Example request

```shell
curl -X GET "https://maxactivity.com/api/v1/events/4336"
  -H "Authorization: Token aaabbbcccdddeeefffggghhhiii"
```

### Request Expectations:

Type | Parameter | Required? | Description
---- | --------- | --------- | -----------
header | `Authorization` | YES | `string`
param | `id` | YES | `integer`

> Response - Success

```shell
status 200 - OK
-H Authorization Token: aaabbbcccdddeeefffggghhhiii

{  
  "event":{  
    "id": 4336,
    "user_id": 1,
    "contact_id": 4,
    "amount": 1000,
    "event_type": "Appointment to Close Life",
    "event_kit_id": null,
    "date": "2017-09-08 12:20:05 AM",
    "created_at": "2017-09-09 12:20:05 AM",
    "updated_at": "2017-09-09 12:20:05 AM"
  }
}
```

### Response Expectations

Type | Key | Success? | Description
---- | --- | -------- | -----------
json | `errors` | NO | `key/value:array<string>`
header | `Authorization Token` | YES | `string`
json | `id` | YES | `integer`
json | `user_id` | YES | `integer`
json | `contact_id` | YES | `integer`
json | `amount` | YES | `string`
json | `event_type` | YES | `integer`
json | `event_kit_id` | YES | `string`
json | `date` | YES | `timestamp`
json | `created_at` | YES | `timestamp`
json | `updated_at` | YES | `timestamp`

## Index

Returns an array of all events viewable by the current user.

### HTTP Request

`GET https://maxactivity.com/api/v1/events`

Only events who the current user is able to access are returned:
Events belonging to the current user or any user lower in the hierarchy.
Events belonging to the current user's trainer are also included.

This endpoint is paginated, meaning only a maximum of `per` events are returned.
By passing in `page`, you can select which group of events are returned.

**Filterable Options**

`last_sync` pass in a timestamp to return only the objects updated LATER than the requested date

`since` pass in a timestamp to return only the objects updated EARLIER than the requested date

`contact_ids` (Array) or `contact_id` (Integer), the endpoint will only return events with the requested contacts.

`user_ids` (Array) or `user_id` (Integer), the endpoint will only return events with the requested users.

`event_ids` to only return events with the passed ids

> Example request

```shell
curl -X GET "https://maxactivity.com/api/v1/events"
  -H "Authorization: Token aaabbbcccdddeeefffggghhhiii"
```

### Request Expectations:

Type | Parameter | Required? | Description
---- | --------- | --------- | -----------
header | `Authorization` | YES | `string`
param | `contact_id` | NO | `integer`
param | `contact_ids` | NO | `array<integer>`
param | `user_id` | NO | `integer`
param | `user_ids` | NO | `array<integer>`
param | `last_sync` | NO | `timestamp`
param | `page` | NO | `integer` (Default: `1`)
param | `per` | NO | `integer` (Default: `25`)
param | `all` | NO | `boolean` (Default: `false`)

> Response - Success

```shell
status 200 - OK
-H Authorization Token: aaabbbcccdddeeefffggghhhiii

{  
  "events":[{}, {}, {}],
  "meta":{}
}
```

### Response Expectations

Type | Key | Success? | Description
---- | --- | -------- | -----------
json | `errors` | NO | `key/value:array<string>`
header | `Authorization Token` | YES | `string`
json | `events` | YES | `array<key/value:eventJson>`
json | `meta` | YES | `key/value:meta`

## Create

Creates a new event for the current user

### HTTP Request

`POST https://maxactivity.com/api/v1/events`

> Example request

```shell
curl -X POST "https://maxactivity.com/api/v1/events"
  -H "Authorization: Token aaabbbcccdddeeefffggghhhiii"
  -d "event[amount]=2000"

# This is the equivalent of:
{
  "event": {
    "amount": 2000
  }
}
```

### Request Expectations:

Type | Parameter | Required? | Description
---- | --------- | --------- | -----------
header | `Authorization` | YES | `string`
param | `event` | YES | `key/value:eventJson`

> Response - Success

```shell
status 201 - Created
-H Authorization Token: aaabbbcccdddeeefffggghhhiii

{  
  "event":{  
    "id": 4336,
    "amount": 2000,
    "..."
  }
}
```

### Response Expectations

Type | Key | Success? | Description
---- | --- | -------- | -----------
json | `errors` | NO | `key/value:array<string>`
header | `Authorization Token` | YES | `string`
json | `event` | YES | `key/value:eventJson`

## Update

Updates a event.

### HTTP Request

`PATCH/PUT https://maxactivity.com/api/v1/events/:id`

Can only update events that belong to the current user

> Example request

```shell
curl -X PATCH "https://maxactivity.com/api/v1/events/4336"
  -H "Authorization: Token aaabbbcccdddeeefffggghhhiii"
  -d "event[amount]=2000"

# This is the equivalent of:
{
  "event": {
    "amount": 2000
  }
}
```

### Request Expectations:

Type | Parameter | Required? | Description
---- | --------- | --------- | -----------
header | `Authorization` | YES | `string`
param | `id` | YES | `integer`
json | `event` | YES | `key/value:eventJson`

> Response - Success

```shell
status 200 - OK
-H Authorization Token: aaabbbcccdddeeefffggghhhiii

{  
  "event":{  
    "id": 4336,
    "amount": 2000,
    "..."
  }
}
```

### Response Expectations

Type | Key | Success? | Description
---- | --- | -------- | -----------
json | `errors` | NO | `key/value:array<string>`
header | `Authorization Token` | YES | `string`
json | `event` | YES | `key/value:eventJson`

## Destroy

Removes a event

### HTTP Request

`DELETE https://maxactivity.com/api/v1/events/:id`

May only delete events that belong to the current user.

> Example request

```shell
curl -X DELETE "https://maxactivity.com/api/v1/events/4336"
  -H "Authorization: Token aaabbbcccdddeeefffggghhhiii"
```

### Request Expectations:

Type | Parameter | Required? | Description
---- | --------- | --------- | -----------
header | `Authorization` | YES | `string`
param | `id` | YES | `integer`

> Response - Success

```shell
status 200 - OK

# No response is returned after deleting an object.
```

### Response Expectations

Type | Key | Success? | Description
---- | --- | -------- | -----------
json | `errors` | NO | `key/value:array<string>`

# Goals

Use this endpoint to return the attributes of goal sets for the current user.

## Show

Returns the data of a single goal set.

### HTTP Request

`GET https://maxactivity.com/api/v1/goal_sets/:id`

This goal set must be accessible from the current user, meaning the goal set must belong to them or one of their downlines.

> Example request

```shell
curl -X GET "https://maxactivity.com/api/v1/goal_sets/4336"
  -H "Authorization: Token aaabbbcccdddeeefffggghhhiii"
```

### Request Expectations:

Type | Parameter | Required? | Description
---- | --------- | --------- | -----------
header | `Authorization` | YES | `string`
param | `id` | YES | `integer`

> Response - Success

```shell
status 200 - OK
-H Authorization Token: aaabbbcccdddeeefffggghhhiii

"goal_set": {
  "id": 215,
  "client_uuid": null,
  "user_id": 1803,
  "title": "My Goals",
  "goals": [
    {
      "id": 2140,
      "client_uuid": null,
      "goal_type": "Contacts Added",
      "percentage_completed": 85.7,
      "goal_amount": 7,
      "current_amount": 6,
      "created_at": "2018-03-15 03:06:25 AM",
      "updated_at": "2018-03-15 03:44:27 AM"
    }
  ],
  "percentage_completed": 85.7,
  "start_date": "2018-03-14",
  "end_date": "2018-03-31",
  "completed_at": null,
  "deleted_at": null,
  "created_at": "2018-03-15 03:06:25 AM",
  "updated_at": "2018-03-15 03:44:27 AM"
}
```

### Response Expectations

Type | Key | Success? | Description
---- | --- | -------- | -----------
json | `errors` | NO | `key/value:array<string>`
header | `Authorization Token` | YES | `string`
json | `id` | YES | `integer`
json | `user_id` | YES | `integer`
json | `title` | YES | `string`
json | `percentage_completed` | YES | `float`
json | `start_date` | YES | `timestamp`
json | `end_date` | YES | `timestamp`
json | `completed_at` | YES | `timestamp`
json | `goals` | YES | `array<Goal>`
json | `created_at` | YES | `timestamp`
json | `updated_at` | YES | `timestamp`
json | `deleted_at` | YES | `timestamp`

## Index

Returns an array of all goal sets viewable by the current user.

### HTTP Request

`GET https://maxactivity.com/api/v1/goal_sets`

Only goal_sets who the current user is able to access are returned:
GoalSets belonging to the current user or any user lower in the hierarchy.

This endpoint is paginated, meaning only a maximum of `per` goal_sets are returned.
By passing in `page`, you can select which group of goal_sets are returned.

**Filterable Options**

`last_sync` pass in a timestamp to return only the objects updated LATER than the requested date

`since` pass in a timestamp to return only the objects updated EARLIER than the requested date

`goal_set_ids` (Array), only return the goal_sets requested.

`user_ids` (Array) or `user_id` (Integer), the endpoint will only return goal_sets belonging to the requested users.

> Example request

```shell
curl -X GET "https://maxactivity.com/api/v1/goal_sets"
  -H "Authorization: Token aaabbbcccdddeeefffggghhhiii"
```

### Request Expectations:

Type | Parameter | Required? | Description
---- | --------- | --------- | -----------
header | `Authorization` | YES | `string`
param | `user_id` | NO | `integer`
param | `user_ids` | NO | `array<integer>`
param | `last_sync` | NO | `timestamp`
param | `page` | NO | `integer` (Default: `1`)
param | `per` | NO | `integer` (Default: `25`)
param | `all` | NO | `boolean` (Default: `false`)

> Response - Success

```shell
status 200 - OK
-H Authorization Token: aaabbbcccdddeeefffggghhhiii

{  
  "goal_sets":[{}, {}, {}],
  "meta":{}
}
```

### Response Expectations

Type | Key | Success? | Description
---- | --- | -------- | -----------
json | `errors` | NO | `key/value:array<string>`
header | `Authorization Token` | YES | `string`
json | `goal_sets` | YES | `array<key/value:goal_setJson>`
json | `meta` | YES | `key/value:meta`

## Create

Creates a new goal_set for the current user

Use `time_period` to automatically set the start and end date for the GoalSet.
When set to Weekly, the start date will be the previous Sunday and the end date will be the following Saturday.
When set to Monthly, the start date will be the previous "month end" and the end date will be the next.

### HTTP Request

`POST https://maxactivity.com/api/v1/goal_sets`

> Example request

```shell
curl -X POST "https://maxactivity.com/api/v1/goal_sets"
  -H "Authorization: Token aaabbbcccdddeeefffggghhhiii"
  -d "goal_set[family_name]=Smith"
  -d "goal_set[given_name]=Sarah"

# This is the equivalent of:
{
  "goal_set": {
    "title": "My Goals",
    "time_period": "weekly",
    "goals": [
      {"goal_type": "Contacts Added", "goal_amount": 7},
      {"goal_type": "Set Appointment", "goal_amount": 20},
      {"goal_type": "Went on Appointment", "goal_amount": 5}
    ]
  }
}
```

### Request Expectations:

Type | Parameter | Required? | Description
---- | --------- | --------- | -----------
header | `Authorization` | YES | `string`
param | `goal_set` | YES | `key/value:goal_setJson`
param | `goals[start_date]` | YES | `timestamp`
param | `goals[end_date]` | YES | `timestamp`

> Response - Success

```shell
status 201 - Created
-H Authorization Token: aaabbbcccdddeeefffggghhhiii

{  
  "goal_set":{  
    "title": "My Goals",
    "start_date": "2018-03-14",
    "end_date": "2018-03-31",
    "goals": [
      {"goal_type": "Contacts Added", "goal_amount": 7},
      {"goal_type": "Set Appointment", "goal_amount": 20},
      {"goal_type": "Went on Appointment", "goal_amount": 5}
    ]
  }
}
```

### Response Expectations

Type | Key | Success? | Description
---- | --- | -------- | -----------
json | `errors` | NO | `key/value:array<string>`
header | `Authorization Token` | YES | `string`
json | `goal_set` | YES | `key/value:goal_setJson`

## Update

Updates a goal_set.

### HTTP Request

`PATCH/PUT https://maxactivity.com/api/v1/goal_sets/:id`

Can only update goal_sets that belong to the current user
Goals can be edited by nesting them

> Example request

```shell
curl -X PATCH "https://maxactivity.com/api/v1/goal_sets/4336"
  -H "Authorization: Token aaabbbcccdddeeefffggghhhiii"
  -d "goal_set[given_name]=Frederick"

# This is the equivalent of:
{
  "goal_set":{  
    "title": "My Goals",
    "start_date": "2018-03-14",
    "end_date": "2018-03-31",
    "goals": [
      {"goal_type": "Contacts Added", "goal_amount": 7},
      {"id": 215, "goal_type": "Set Appointment", "goal_amount": 20},
      {"id": 216, "_destroy": true}
    ]
  }
}
```

### Request Expectations:

Type | Parameter | Required? | Description
---- | --------- | --------- | -----------
header | `Authorization` | YES | `string`
param | `id` | YES | `integer`
json | `goal_set` | YES | `key/value:goal_setJson`

> Response - Success

```shell
status 200 - OK
-H Authorization Token: aaabbbcccdddeeefffggghhhiii

{  
  "goal_set":{  
    "title": "My Goals",
    "start_date": "2018-03-14",
    "end_date": "2018-03-31",
    "goals": [
      {"goal_type": "Contacts Added", "goal_amount": 7},
      {"id": 215, "goal_type": "Set Appointment", "goal_amount": 20},
      {"id": 216, "_destroy": true}
    ]
  }
}
```

### Response Expectations

Type | Key | Success? | Description
---- | --- | -------- | -----------
json | `errors` | NO | `key/value:array<string>`
header | `Authorization Token` | YES | `string`
json | `goal_set` | YES | `key/value:goal_setJson`

## Destroy

Removes a goal_set

### HTTP Request

`DELETE https://maxactivity.com/api/v1/goal_sets/:id`

May only delete goal_sets that belong to the current user.

> Example request

```shell
curl -X DELETE "https://maxactivity.com/api/v1/goal_sets/4336"
  -H "Authorization: Token aaabbbcccdddeeefffggghhhiii"
```

### Request Expectations:

Type | Parameter | Required? | Description
---- | --------- | --------- | -----------
header | `Authorization` | YES | `string`
param | `id` | YES | `integer`

> Response - Success

```shell
status 200 - OK

# No response is returned after deleting an object.
```

### Response Expectations

Type | Key | Success? | Description
---- | --- | -------- | -----------
json | `errors` | NO | `key/value:array<string>`
