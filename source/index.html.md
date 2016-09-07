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
And then: ./deploy.rb
-->

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

`GET https://zygygames.com/api/login`
`GET http://staging.zygygames.com/api/login`

### Request expectations:

> Example request

```shell
curl -X GET "https://zygygames.com/api/login"
curl -X GET "http://staging.zygygames.com/api/login"
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

{
  "error": "Invalid Username or Password"
}
```

> Response - Success

```shell
-H Authorization-Success : true
-H Authorization-Code : OUKdeYQf1qiEoTF8clnk

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

`GET https://zygygames.com/api/register`
`GET http://staging.zygygames.com/api/register`

### Request expectations:

> Example request

```shell
curl -X GET "https://zygygames.com/api/register"
curl -X GET "http://staging.zygygames.com/api/register"
  -H "App-Identifier: 123456789-987654321"
  -d "username=xxxxx&referral_code=xxxxxx&first_name=xxxxx&last_name=xxxxx&email=xxxxx@xxx.com&confirm_email=xxxxx@xxx.com&birthday=xx/xx/xx&password=xxxxxxxx&confirm_password=xxxxxxxx&accepted_tos=true"
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
param | `accepted_tos` | true | Checkbox confirming user agrees to TOS and Privacy Policy

### Response Expectations

> Response - Failure

```shell
-H Authorization-Success : false
-H Error-Message : Username already taken.

{
  "error": "Username already taken."
}
```

> Response - Success

```shell
-H Authorization-Success : true
-H Authorization-Code : OUKdeYQf1qiEoTF8clnk

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

# Games

Returns a json array of every game.

Only display the games with `published: true` to the user.

The `Games` endpoint does not require authorization.

## Endpoint

`GET https://zygygames.com/api/games`
`GET http://staging.zygygames.com/api/games`

### Request Expectations

> Example request

```shell
curl -X GET "https://zygygames.com/api/games"
curl -X GET "http://staging.zygygames.com/api/games"
```

### Response Expectations

> Response

```shell

[
  {
    "name":"Zygy App",
    "updated_at":"2016-06-15T04:15:33.062Z",
    "how_to_description":null,
    "game_description":null,
    "app_store_id":null,
    "published":false,
    "app_icon":"/system/games/app_icons/missing.png"
  },
  {
    "name":"Stack Em",
    "updated_at":"2016-06-15T04:21:35.661Z",
    "how_to_description":"",
    "game_description":"Build them as high as you can.",
    "app_store_id":null,
    "published":false,
    "app_icon":"http://s3.amazonaws.com/zygy-user-image-uploads/games/app_icons/000/000/004/original/Stack-Em_Icon-20.png?1465950356"
  },
  {
    "name":"Pirate Pike",
    "updated_at":"2016-06-15T04:38:01.771Z",
    "how_to_description":"User your finger to trace \u0026 create bungee bridges. Collect coins and magical boosters and beware of cannon balls!",
    "game_description":"Help Pike the Pirate on his adventurous journey collecting loot across the skies of the Isles of Bounty.",
    "app_store_id":null,
    "published":true,
    "app_icon":"/system/games/app_icons/missing.png"
  }
]
```

Type | Key  | Description
---- | ---  | -----------
json | `name` | Name of the game
json | `updated_at` | The last time the game was updated
json | `game_description` | Description of what the game is
json | `how_to_description` | How to play the game
json | `app_store_id` | If the game is available on the App Store, the ID used to identify it.
json | `published` | Whether or not the game is published and should be shown to the user.
json | `app_icon` | The url to the app's icon

# Zygy Now

Zygy Now is a a Scoreboard based tracker that allows a User to track their information. It contains information for the selected month, as well as an 'all-time best'.

A user can select which month to view, and can also change the user in which information is shown. A user can ONLY view a different user's information if the perspective user is a 'downline' of the current user.

All values and dates sent as a response from the server will be pre-formatted for display.

## Endpoint

`GET https://zygygames.com/api/zygy_now`
`GET http://staging.zygygames.com/api/zygy_now`

### Request Expectations

> Example request

```shell
# params are optional
curl -X GET "https://zygygames.com/api/zygy_now"
curl -X GET "http://staging.zygygames.com/api/zygy_now"
  -H "App-Identifier: 123456789-987654321"
  -H "Authorization-Code: OUKdeYQf1qiEoTF8clnk"
  -d "user_identification=rockster160&month=4&year=2016"
```

Type | Parameter | Required? | Description
---- | --------- | --------- | -----------
header | `App-Identifier` | true | App Identification String
header | `Authorization-Code` | true | User Authorization Code
params | `user_identification` | false | `Default: current user` If supplied, looks up user by this value. (Username, Email, or Zygy ID)
params | `month` | false | `Default: current month` If supplied, filters dates based on this integer value. (1=January, 12=December)
params | `year` | false | `Default: current year` If supplied, filters dates based on this integer value. (YYYY)

### Response Expectations

> Response - Failure

```shell
-H Authorization-Success : false
-H Error-Message : Cannot view a user unless they are on your team.

{
  "error": "Cannot view a user unless they are on your team."
}
```

> Response - Success

```shell
-H Authorization-Success : true

{
  "current_month":"06/16",
  "data":[  
    {  
      "name":"New Players Added",
      "current_month_value":"0",
      "best_month":"04/2016",
      "best_month_value":"5"
    },
    {  
      "name":"Revenue",
      "current_month_value":"0",
      "best_month":"01/2016",
      "best_month_value":"$0.00"
    },
    {  
      "name":"Pirate Pike",
      "current_month_value":"0",
      "best_month":"02/2016",
      "best_month_value":"20,143"
    },
    {  
      "name":"Kubed",
      "current_month_value":"0",
      "best_month":"05/2016",
      "best_month_value":"828"
    }
  ]
}
```

Type | Key | Success? | Description
---- | --- | -------- | -----------
header | `Authorization-Success` | N/A | Stringified boolean representing whether or not the request was successful.
header | `Error-Message` | false | Same message as json-error, passed as a Header.
json | `error` | false | Description of why a failure resulted - Displayable to user
json | `current_month` | true | The currently scoped month as a String for display
json | `data` | true | Array of 'data rows' containing values for each section.
json | `name` | true | The name of either the game or the type of data the row contains
json | `current_month_value` | true | String value of row at selected month
json | `best_month` | true | String value of date the `best_month_value` occurred.
json | `best_month_value` | true | String value of row at best month

# News Feed

News Feed is a list of articles that contain basic information shown to users.

The News Feed will return a Webview, and should be rendered as is, using the standard App's Navigation.

That means showing a back button and the normal tab view that is displayed on other pages.

## Endpoint

`GET https://zygygames.com/api/news_feed`
`GET http://staging.zygygames.com/api/news_feed`

### Request Expectations

> Example request

```shell
# params are optional
curl -X GET "https://zygygames.com/api/news_feed"
curl -X GET "http://staging.zygygames.com/api/news_feed"
  -H "App-Identifier: 123456789-987654321"
  -H "Authorization-Code: OUKdeYQf1qiEoTF8clnk"
  -d "page=4&per=2"
```

Type | Parameter | Required? | Description
---- | --------- | --------- | -----------
header | `App-Identifier` | true | App Identification String
header | `Authorization-Code` | true | User Authorization Code

### Response Expectations

> Response - Failure

```shell
-H Authorization-Success : false
-H Error-Message : Authorization failure.

{
  "error": "Authorization failure."
}
```

> Response - Success

```shell
<!DOCTYPE html>
<html>
 <!-- The News feed with associated styles should be here. -->
</html>
```
