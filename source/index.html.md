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

All of the examples will be shown in `JSON` (for responses FROM the server)

This is to retain a consistent syntax, as objects are handled in JSON. Formatting and parsing the parameters is up to the client.

## Basic details about requests

> Successful Response Status Codes

```shell
status 200 - OK
# This status states that everything went according to plan and things are good.
# Generally this status will be returned with a GET request, assuming all things went well.

status 201 - Created
# This status states that an object was successfully created.
# This will generally be the response for a POST request.

status 204 - No Content
# This status states that an object was successfully manipulated.
# This is generally the response for PUT/PATCH and DELETE requests.
```

> Failure Response Status Codes

```shell

status 400 - Bad Request
{
  "errors": ["param is missing or the value is empty: user"]
}
# This error states that something went wrong with parsing the params that were passed in. Generally, it's because the params were not nested under the resource name that they are updating.
# These should NOT be shown to the user and should be handled like a crash.

status - 401 Unauthorized
{
  "errors": ["Invalid email or password."]
}
# This error can occur whenever a user enters a bad email/password upon logging in, or when a password change causes their authorization token to become invalid.
# These errors ARE safe to display to the user.

status - 403 Forbidden
{
  "errors": ["You are not authorized to do that."]
}
# This error occurs if you attempt to make a request that the current user does not have access to- for example, attempting to view somebody not on their team.
# These errors ARE safe to display to the user.

status 500 - Server Error
{
  "errors": ["Something went wrong."]
}
# If a 500 is returned, it means something went wrong on the server side. This should be reported, when possible as it generally means there is a crash.
# These errors are NOT safe to display to the user and should be handled like a crash.

status 502 - Bad Gateway (Could not connect to server)
# (No response)
# This error generally occurs while the server is restarting, but informs the app that the server could not be reached for some reason or another.
# In these cases, the request should be tried again, or a message should be displayed to the user to try again later.
```

The basic response codes are shown on the right. These status codes are the HTTP statuses that are made.
Generally, when interacting with maxactivity.com, any time something goes wrong, you will receive a response in the form of a key/value/dictionary/associative array with a single key, `errors`. The value for that key is an array of human-readable messages. Errors will always be returned as an array, although there will usually one be 1 error present.
**Error messages not in the range of 5xx are safe to display to the user.**

* 1xx: **Information** (Unused)
* 2xx: **Success** - Everything went according to plan
* 3xx: **Redirect** - The url you hit is no longer officially supported and you are being automatically redirected to a different url.
* 4xx: **Client Error** - Something was wrong with the request- generally this means you forgot a required parameter, but could also be an invalid url, requesting an object you don't have permission to view, or otherwise doing something outside of your permissions.
* 5xx: **Server Error** - Something went wrong with the server. This could be a bug in the code or that the server is down. Generally these mean something is going REALLY wrong.

You can read more about status codes [Here](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes)

## Formats

`timestamp`s are always returned as strings in UTC with the format:

`YYYY-MM-DD HH:MM:SS P`

Example: `2018-01-09 08:30:47 PM`

> Example Response

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

# RESTful API

maxactivity.com uses RESTful APIs.
RESTful means we adhere to "Representational State Transfer" constraints.
This essentially means you interact with the API using CRUD:
`Create`
`Read`
`Update`
`Destroy`

All controllers conform to a pretty simple syntax with 6 different HTTP methods.

The below methods will show endpoints. The word `resources` in the url should be replaced with the *pluralized, lowercase, underscored* separated object name. For example, `users` or `month_end` depending on the object you're using.
Any word that starts with `:` means it's a required parameter that's included in the url.
For example:
`/resources/:id`

Means your url should look like this: `https://maxactivity.com/api/v1/users/123`
The `123` is the *server id* for that user.

## SHOW

> Example Response

```shell
{
  "user": {
    "id": 1803,
    ...
  },
  "meta": {}
}
```

GET `/resources/:id`

Returns the resource by itself under the **singular** key of it's objects name.

This resource must be *viewable* by the current user, or you will get a `404 - Not Found` error.

.

.

.

.

.

## INDEX

> Example Response

```shell
{
  "users": [{}, {}, {}],
  "meta": {
    "total_count": 25,
    "total_pages": 1,
    "page": 1,
    "next_page": null,
    "prev_page": null,
    "per": 25
  }
}
```

GET `/resources`

This endpoint will return the *paginated* objects that the current user is permitted to view.

*Pagination*

All requests that return more than a single resource will be paginated. Pagination means the results are given in "pages", to prevent a massive request being sent when only a few results are desired.

*Request*

There are 3 key parameters in pagination. `page`, `per`, and `all`

Defaults:

Param | Default | Description
---- | --------- | ---------
page | `1` | Which "page" of objects to return.
per | `25` | How many objects to return per page.
all | `false` | When set to `true`, pagination is not applied and all results are returned. (Not recommended unless you know the request does not have many objects.)

For example, when `page: 2, per: 50` is given in the parameters, you will receive objects in the list between indices `51-100` (Page 1 would be `1-50` since `per` is set to return 50 objects at a time)

Additionally, most objects have basic filters. Filters are only applied on the objects the current user is allowed to see, so they cannot be used to view objects not normally viewable.
Some endpoints have additional filters which will be covered in the specific resource sections.

`last_sync` pass in a timestamp to return only the objects updated LATER than the requested date.

`since` pass in a timestamp to return only the objects updated EARLIER than the requested date. (Can be used in conjuction with `last_sync` to only get objects created in a specific time period)

`user_ids` pass this with an array of ids (or a single id) to get the objects from the endpoint that belong directly to the users matching those ids.

`resource_ids` pass this with an array of ids (or a single id) to get the objects from the endpoint with those ids.

*Response*

Any time you make a request to an endpoint that returns paginated objects, the `meta` data in your response will contain details about the paginated objects.


## CREATE

> Example Request

```shell
{
  "resource": {
    "client_uuid": "aaaa-bbbb-cccd-dddd"
    "details": "stuff about the resource",
    "nested_resource": [
      {"details": "Things"},           # This will create a new nested object
      {"id": 123, "details": "Stuff"}, # This will update an existing nested object
      {"id": 321, "_destroy": true}    # This will destroy an existing object
    ]
  }
}
```

POST `/resources`

This endpoint is used to create a new object. *Objects created this way will automatically belong to the current user.*

This endpoint will often have required properties to be included for the object to be created properly. Required properties will be outlined in each objects specific documentation.

The `client_uuid` is a string created by the client/app. This string is ONLY used so that the app can create a temporary object to display before the object is returned from the server, at which point the included `client_uuid` will be included to match the objects together.

Occasionally, objects will be nested within another object, such as `notes` under `contacts`, or `goals` under `goal_sets`. This is technically bad practice and not RESTful.

In these cases, the objects should be nested as a property in the object they are under, with the key being the *pluralized, lowercase, underscored* name of the object. If an `id` for the nested resource is not included, a new resource will be created, otherwise the resource will be updated. If the special key `_destroy` is passed with a value of `true`, the nested object will be deleted.

## UPDATE

> Example Request

```shell
{
  "resource": {
    "name": "Frederick",
  }
}
```

PUT/PATCH `/resources/:id`

This is the endpoint used to modify an object. *A user can only modify objects that belong directly to them.*

Only send the properties that are currently being modified- this prevents race conditions where the client updates a value that was since modified on the server.

All properties must be nested beneath the *singular, lowercase, underscored* resource key.

## DESTROY

DELETE `/resources/:id`

No other information is required for this endpoint.

Submitting to this endpoint will destroy the object. *A user can only destroy objects that belong directly to them.*

# Authentication

**ALL** endpoints require that the user be authenticated, unless otherwise specified. There are 2 ways to authenticate:

1. Authorization Token (Recommended)
2. Username/Password combination (Not recommended)

## Authorization Token

An authentication token will be returned in the *headers* of any request where the user was successfully authenticated.
This token is found as the value for the head "Authorization Token" and can be stored in the device's local storage safely.
In order to authenticate, pass the `Authorization` header in your request with a value of `Token aabbccdd` where `aabbccdd` is the authorization retrieved from the previous header.

## Username/Password combination

In order to get an Authorization Token, a user must first log in with their username or email and password combination.
These should **NOT** be stored on the device, as the device's local memory can potentially be accessed.
In order to authenticate, pass the `Authorization` header in your request with a value of `Basic aabbccdd` where `aabbccdd` is the Base64 encoded string of the users's email and password joined together with a colon. For example: `user@email.com:password` becomes `dXNlckBlbWFpbC5jb206cGFzc3dvcmQ=` when encoded, so the Header you would pass would be `Authorization: Basic dXNlckBlbWFpbC5jb206cGFzc3dvcmQ`.

If successful, the `Authentication Token` will be returned in the response headers.

Any time that the server responds with a `401` status code, the user should be logged out of the current session and asked to sign in again. The only time this should happen is if they change their password. A "log out" should entail clearing all local memory of the user, including the stored `Authorization Token`.

## Login

> Example Request

```shell
POST /login

{
  "email": "user@email.com", # * Required
  "password": "password"     # * Required
}
```

> Successful Response

```shell
Header: "Authorization Token": "aaabbbcccdddeeefffggghhhiii"
```

<aside class="notice">This endpoint does NOT require authentication.</aside>

This endpoint authorizes a user.

<aside class="warning">
Remember — Store the `Authorization Token` in-app and over-write it on every request! If the user changes their password, that token is regenerated. Not saving this token will cause your user to be unauthenticated.
</aside>

## Password Reset

> Example request

```shell
POST /password_reset

{
  "email": "user@email.com"
}
{
  "email": "xupr7"
}
```

<aside class="notice">This endpoint does NOT require authentication.</aside>

This endpoint is used to trigger a password reset email for users who have forgotten their password

You can submit an email address OR a solution number. If found in the database, we will deliver a password reset to the email we have on file for that particular user.

This endpoint will ALWAYS return a 200, regardless of if the email is actually sent or the email actually exists within our database. This is a common principle to prevent malicious users from sending batches of emails and figuring out which emails we have in our database.

# Updates / Websocket

## Updates

> Example request

```shell
GET /updates

{
  "last_sync": "2018-02-01 04:00:00 PM"
}
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

This endpoint requires a `last_sync` parameter to be sent, and will respond with all viewable objects that have been updated or deleted since the requested timestamp.

The response from this endpoint is a complex object. At the top level there are 3 keys: `changed` (Array), `deleted` (Array), `timestamp` (Timestamp/String)

The `timestamp` is the current time when the request was made. (This can/should be stored for later)
The `changed` and `deleted` arrays contain the objects that were changed or deleted, respectively.
The objects are key/value pairs where the key is the object type and the value is an array of the objects of that type.

<aside class="warning">
Because this endpoint returns ALL changes, it's possible for the request to be very large.

If a significant amount of time has passed, it's recommended to instead unauthenticate the user or do a complete/manual resync.
</aside>

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
{
  "command": "subscribe",
  "identifier": "{\"channel\":\"GlobalChannel\"}"
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

## Connect

The initial connection must be made to the url:
`wss://maxactivity.com/cable`
The request origin must be set to the root domain,
`https://maxactivity.com`

In order to use the websockets, you must be authorized. You can authorize the same way as other endpoints, by including the `Authorization` header.

<aside class="warning">
If a connection is lost, the app should continually attempt to reconnect to the server until the connection is once again made. At that point, you must also subscribe again.
</aside>

## Subscribe

In order to actually get data back from the server, you must first subscribe to a channel.

The connection is like having a cell phone network, but subscribing to a channel is actually dialing a phone number.

There are 2 channels you must subscribe to: `GlobalChannel` and `UpdatesChannel`
The `UpdatesChannel` is specific to the current user. This channel will include relevant details about changed objects.
The `GlobalChannel` contains information for all users. It will be used for cases such as the `month_end` date being changed.

In order to subscribe, you must send data through the websocket telling the server which channel you want to subscribe to.

`command` tells the server what you want to do- `subscribe`.

`identifier` is a **String** containing the subscription data. Specifically, in that string, you want to pass the `channel` you are subscribing to and the `channel_id`, which is a string telling the server which user you want updates for.

The `channel` is a hard set string as `UpdatesChannel` or `GlobalChannel`
The `channel_id` should only be included with the `UpdatesChannel` and the value is a string `updates_1111` - replace `1111` with the current user's server id.

## Response

The response from the server will not come immediately, but as a message through the websocket. The message will always be a JSON packet. This data will exactly match the format of the `updates` endpoint, but typically only a few objects will be sent at a time and they will be sent as soon as those objects are changed on the server.

## Special Cases

> Example message

```shell
{
  "timestamp": "2018-03-17 06:35:50 PM",
  "changed": {
    "users": [{}, {}]
  },
  "deleted": {
    "contacts": [{}, {}]
  },
  "meta": {
    "month_end": "2018-04-02 11:00:00 PM",
    "alert": {
      "title": "Month End has changed!",
      "message": "You now have an extra 5 hours to get your stuff in."
    },
    "function": "resync"
  }
}
```

Websockets will always contain 4 keys. `timestamp`, `changed`, `deleted`, and `meta`
The `timestamp` is the server time that the message was sent. It's recommended to store this value client side and use this value when requesting from the `updates` endpoint.
The `changed` key will contain a dictionary where the keys will be object types, and the values are arrays containing the changed objects.
The `deleted` key will contain any items that were deleted. These should be handled accordingly app-side.
The `meta` key will **always** contain the current month end date. It can also contain values that should be handled app side.

These can be:
`alert` will contain a `title` and `message`. These should display an alert popup to the user with the corresponding values that require the user to dismiss before disappearing.
`function` will contain a string that represents a command.
When the command is `resync` the app should drop the current database and sync again using the standard endpoints. This is used when a user's company is changed, causing all values within the app to potentially change. It can also be used when debugging.
When the command is `logout` a user should **immediately** be logged out- the database should be dropped and the `Authentication Token` cleared. This will generally be used if there is suspicious activity on the user's account.

# Endpoints

## Account

### Month End

> Example request

```shell
GET /account/month_end

{
  "month_end": "2018-02-01 04:00:00 PM"
}
```

<aside class="notice">This endpoint does NOT require authentication.</aside>

This endpoint simply returns the timestamp for the next month-end date.

You can optionally add a `date` parameter to get the month end for a specific date.

<aside class="notice">
The Month End date is calculated as the first week day in the following month, and skips Labor Day and New Years Day.

Month End can occasionally be updated to be a different date/time for various reasons.
</aside>

### OneSignal Notifications

> Example request

```shell
POST /account/subscribe_notifications

{
  "onesignal_id": "asdfasdf-asdfasdf-asdfasdf"
}
```

Once the user has subscribed to OneSignal Notifications, send their OneSignal ID to this endpoint in order to allow remote notifications from the server.

## User

> Example Response

```shell
{
  "user": {
    "id": 1803,
    "client_uuid": "aaaa-bbbb-cccc-dddd",
    "upline_id": 4534,
    "trainer_id": 3533,
    "downline_ids": [4585,4589,4586],
    "email": "rocco@zygy.com",
    "family_name": "Nicholls",
    "given_name": "Rocco",
    "phone_number": "(385)259-9640",
    "profile_picture_url": "",
    "state": "UT - Utah",
    "solution_number": "xupr7",
    "is_rvp": false,
    "created_at": "2017-03-05 04:52:03 AM",
    "updated_at": "2018-03-18 12:01:34 AM",
    "current_speed": 9,
    "breakdown": {
      "ROLLING 7 DAY NUMBERS": {
        "New Prospects": 9,
        "Appointments Set": 2,
        "Appointments Done": 0,
        "Invites": 0,
        "New Shows": 0,
        "Upcoming Appointments": 0
      },
      "TOTALS": {
        "People Working With": 10,
        "Qualified People": 0,
        "Prospects": 9
      }
    }
  },
  "meta": {}
}
```

> Example Request

```shell
{
  "client_uuid": "aaaa-bbbb-cccc-dddd",
  "upline_id": 1,
  "trainer_id": 2,
  "email": "user@email.com", # * Required on first create
  "password": "password",    # * Required on first create
  "given_name": "Rocco",
  "family_name": "Nicholls",
  "phone_number": "(385) 259-9640",
  "profile_picture": <file>,
  "state": "UT - Utah",
  "solution_number": "xupr7",
  "is_rvp": true
}
```

<aside class="notice">Creating a user does NOT require authentication. Use the Create User endpoint to register a new account.</aside>

## Company

> Example Response

```shell
{
  "company": {
    "id": 1,
    "client_uuid": "aaaa-bbbb-cccc-dddd",
    "name": "None",
    "event_groups": {
      "Appointment": [
        "Set Appointment",
        "Went on Appointment"
      ],
      "Other": [
        "Invited to Meeting",
        "Went to Meeting"
      ]
    },
    "rating_groups": {
      "Client Rating": [
        "Has Kids",
        "Homeowner",
        "Income Over 40k",
        "Married",
        "Age 25-55"
      ],
      "Recruit Rating": [
        "Competitive",
        "Credible",
        "Hungry",
        "Motivated",
        "People Skills"
      ]
    },
    "goal_types": [
      "Contacts Added",
      "Set Appointment",
      "Went on Appointment",
      "Invited to Meeting",
      "Went to Meeting"
    ],
    "goal_time_periods": [
      "Weekly",
      "Monthly"
    ],
    "meeting_types": [
      "Training Meeting",
      "Opportunity Meeting",
      "Emergency Meeting"
    ]
  },
  "meta": {}
}
```

<aside class="notice">
This endpoint is used singularly.

`GET /api/v1/company`

Additionally, only the SHOW action exists for this endpoint. A user cannot view other companies or modify any company details.
</aside>

The details provider here should be used to build the basic structure of the objects that are specific to each Company.
By default, the "None" company will be used until a user is added to a different company.

`event_groups` are used to filter different types of events.
Inside of the event groups are `event_types` which are the names of the specific events that can occur.

`rating_groups` are the properties on Contacts that can be set, which demonstrate how valuable the Contact is to the company as either a customer or representative.
`rating_types` are the specific strings that are name the values set.

`goal_types` are generally the event types plus additional goals that can be created server side.

The above values will be needed when creating objects relative to each category.

## Contact

> Example Response

```shell
{
  "contact": {
    "id": 96768,
    "client_uuid": "aaaa-bbbb-cccc-dddd",
    "user_id": 4595,
    "family_name": "Tact",
    "given_name": "Cont",
    "phone_number": "111-111-1111",
    "street1": null,
    "street2": null,
    "city": null,
    "state": null,
    "zip": null,
    "country": null,
    "ratings": { # These values are dependent on the Company
      "Client Rating": {
        "Has Kids": false,
        "Homeowner": false,
        "Income Over 40k": false,
        "Married": false,
        "Age 25-55": false
      },
      "Recruit Rating": {
        "Competitive": false,
        "Credible": false,
        "Hungry": false,
        "Motivated": false,
        "People Skills": false
      }
    },
    "client_rating": null,  # The names do not change, but these tie to the corresponding values from the company rating groups
    "recruit_rating": null, # The names do not change, but these tie to the corresponding values from the company rating groups
    "credibility_rating": null,
    "imported": null,
    "recruited": false,
    "invited": false,
    "sold": false,
    "attended": false,
    "list_ids": [],
    "created_at": "2018-03-17 10:58:59 PM",
    "updated_at": "2018-03-17 10:58:59 PM",
    "notes": [
      {"id": 123, "content": "This is a note"}
    ]
  },
  "meta": {}
}
```

> Example Request

```shell
{
  "contact" {
    "client_uuid": "aaaa-bbbb-cccc-dddd",
    "family_name": "Tact",
    "given_name": "Cont",
    "phone_number": "111-111-1111",
    "street1": "some street",
    "street2": "some street",
    "city": "some city",
    "state": "some state",
    "zip": "some zip",
    "country": "some country",
    "ratings": { # These values are dependent on the Company
      "Client Rating": {
        "Has Kids": false,
        "Homeowner": false,
        "Income Over 40k": false,
        "Married": false,
        "Age 25-55": false
      },
      "Recruit Rating": {
        "Competitive": false,
        "Credible": false,
        "Hungry": false,
        "Motivated": false,
        "People Skills": false
      }
    },
    "credibility_rating": 5,
    "notes": [
      {"id": 123, "content": "This is a note"}
    ]
  }
}
```

<aside class="notice">Notes are treated as a nested resource for Contacts. To modify notes, you must do so through the nested resources syntax.</aside>

## Note

> Example Response

```shell
{
  "note": {
    "id": 6088,
    "client_uuid": null,
    "contact_id": 37140,
    "content": "Some Stuff",
    "created_at": "2018-03-20 01:02:07 AM",
    "updated_at": "2018-03-20 01:02:07 AM"
  }
}
```

> Example Request

```shell
{
  "note": {
    "client_uuid": null,
    "contact_id": 37140, # * Required on first create
    "content": "Some Stuff"
  }
}
```

### Additional Filters

`contact_id` can be passed as an array or a single integer, will return only notes belonging to those Contact(s)

## List Contact

> Example Response

```shell
{
  "list_contacts": [
    {
      "list_id": 26704,
      "contact_id": 37140,
      "created_at": "2018-02-28 05:57:42 AM",
      "updated_at": "2018-02-28 05:57:42 AM",
      "deleted_at": null
    },
    {
      "list_id": 10816,
      "contact_id": 96766,
      "created_at": "2018-03-15 03:54:31 AM",
      "updated_at": "2018-03-15 03:54:31 AM",
      "deleted_at": null
    },
    ...
  ],
  "meta": {
    "total_count": 22,
    "total_pages": 1,
    "page": 1,
    "next_page": null,
    "prev_page": null,
    "per": 25
  }
}
```

<aside class="notice">This resource only has an INDEX endpoint.</aside>

## List

> Example Response

```shell
{
  "list": {
    "client_uuid": "aaaa-bbbb-cccc-dddd",
    "id": 33634,
    "user_id": 4585,
    "name": "Top 25 for Recruiting",
    "contact_ids": [33134, 33244, 54334],
    "created_at": "2018-03-17 09:47:57 PM",
    "updated_at": "2018-03-17 09:47:57 PM"
  }
}
```

> Example Request

```shell
{
  "list": {
    "client_uuid": "aaaa-bbbb-cccc-dddd",
    "name": "Top 25 for Recruiting", # * Required on first create
    "contact_ids": [33134, 33244, 54334],
    "add_contact_ids": [123]
    "remove_contact_ids": [33134]
  }
}
```
When creating or updating a list, Contacts can be modified in 3 ways:

1. `contact_ids` - This sets the new Contact list. Any contacts not included will be removed, and extra contacts will be added. Existing contacts will be unchanged. (Not recommended)
2. `add_contact_ids` - Adds new contacts by id without modifying existing contacts.
3. `remove_contact_ids` - Removes contacts from a list if they are present.

### Additional Filters

`contact_id` can be passed as an array or a single integer, will return only lists which include those Contact(s)

## Event

> Example Response

```shell
{
  "events": {
    "id": 28655,
    "client_uuid": "aaaa-bbbb-cccc-dddd",
    "meeting_id": null,
    "user_id": 1803,
    "contact_id": null,
    "amount": null,
    "event_group": "Appointment",    # Corresponds to Company Event Group
    "event_type": "Set Appointment", # Corresponds to Company Event Type
    "event_kit_id": null,
    "date": "2018-03-03 07:00:54 PM",
    "created_at": "2018-03-04 09:10:21 PM",
    "updated_at": "2018-03-04 09:10:21 PM"
  }
}
```

> Example Request

```shell
{
  "events": {
    "client_uuid": "aaaa-bbbb-cccc-dddd",
    "meeting_id": null,
    "meeting_schedule_id": null,
    "meeting_date": null,
    "contact_id": 33134,             # * Required on first create
    "amount": null,
    "event_group": "Appointment",    # * Required on first create
    "event_type": "Set Appointment", # * Required on first create
    "event_kit_id": null,
    "date": "2018-03-03 07:00:54 PM"
  }
}
```

When creating Events, can optionally add a meeting. If a meeting is added, the Contact will be expected to attend the meeting.

The meeting can be set one of 2 ways:

1. Pass the `meeting_id`, if accessible.
2. Pass the `meeting_schedule_id` AND the `meeting_date` - This will automatically set the corresponding meeting id.

## Goal Set

> Example Response

```shell
{
  "goal_set": {
    "id": 215,
    "client_uuid": "aaaa-bbbb-cccc-dddd",
    "user_id": 1803,
    "title": "My Goals",
    "percentage_completed": 100,
    "start_date": "2018-03-14",
    "end_date": "2018-03-31",
    "completed_at": "2018-03-15 04:00:23 AM",
    "deleted_at": null,
    "created_at": "2018-03-15 03:06:25 AM",
    "updated_at": "2018-03-15 04:00:31 AM"
  }
}
```

> Example Request

```shell
{
  "goal_set": {
    "client_uuid": "aaaa-bbbb-cccc-dddd",
    "title": "My Goals",
    "start_date": "2018-03-14",        # * Required on first create
    "end_date": "2018-03-31",          # * Required on first create
    "time_period": "weekly"            # * Alternative to the start/end date
  }
}
```

<aside class="notice">`time_period` can be included instead of `start_date` and `end_date`. Valid options are: "weekly" and "monthly". When passed, the time stamps are automatically determined. Weekly is Sunday to Sunday, monthly is Month End to Month End.</aside>


## Goal

> Example Response

```shell
{
  "goal": {
    "id": 2139,
    "client_uuid": null,
    "goal_type": "Contacts Added",
    "goal_set_id": 214,
    "goal_set_client_uuid": null,
    "percentage_completed": 128.6,
    "goal_amount": 7,
    "current_amount": 9,
    "created_at": "2018-03-15 03:02:12 AM",
    "updated_at": "2018-03-15 03:54:31 AM"
  }
}
```

> Example Request

```shell
{
  "goal": {
    "client_uuid": null,
    "goal_type": "Contacts Added",
    "goal_amount": 7,
    "goal_set_id": 214 # * Required on first create
  }
}
```

## Meeting Schedule

> Example Response

```shell
{
  "meeting_schedule": {
    "id": 1,
    "client_uuid": "aaaa-bbbb-cccc-dddd",
    "orchestrator_id": 3485,
    "meeting_type": "Training Meeting",
    "location": "7070 Stone Dr Daphne AL",
    "description": "Saturday Training",
    "schedule": "DTSTART;TZID=MDT:20180312T010000\nRRULE:FREQ=WEEKLY;UNTIL=20180430T093000Z",
    "repeats": true,
    "repeat_type": "weekly",
    "start_date": "2018-03-12",
    "end_date": "2018-04-30",
    "start_time": "7:00",
    "end_time": "9:30"
  }
}
```

> Example Request

```shell
{
  "meeting_schedule": {
    "client_uuid": "aaaa-bbbb-cccc-dddd",
    "meeting_type": "Training Meeting", # * This must correspond to the CompanyMeetingTypes
    "location": "7070 Stone Dr Daphne AL",
    "description": "Saturday Training",
    "repeat_type": "weekly", # Can be null for non-repeating meeting
    "starts_at": "2018-03-12 7:00:00 PM", # Alternative to setting date and time separately
    "ends_at": "2018-04-30 9:30:00 PM", # Alternative to setting date and time separately
    "start_date": "2018-03-12",
    "end_date": "2018-04-30",
    "start_time": "7:00",
    "end_time": "9:30"
  }
}
```

Start and End dates and times can be set by using the `*_at` properties instead of both the `*_date` and `*_time`.

The RVP or user that creates the meeting schedule is referred to as the `orchestrator`

## Meeting

> Example Response

```shell
{
  "meeting": {
    "id": null,
    "client_uuid": null,
    "meeting_schedule_id": 1,
    "original_date": "2018-03-11",
    "starts_at": "2018-03-12 07:00:00 AM",
    "ends_at": "2018-03-12 09:30:00 AM",
    "location": "7070 Stone Dr Daphne AL",
    "description": "Saturday Training"
  },
  "meta": {}
}
```

> Example Request

```shell
{
  "meeting_schedule_id": 1, # * Required every time, NOT nested in the meeting.
  "meeting": {
    "client_uuid": null,
    "starts_at": "2018-03-12 07:00:00 AM",
    "ends_at": "2018-03-12 09:30:00 AM",
    "location": "7070 Stone Dr Daphne AL",
    "description": "Saturday Training"
  }
}
```

<aside class="notice">This resource only has SHOW, UPDATE, and DESTROY endpoints.</aside>

When using the SHOW endpoint, instead of passing the server id, instead, pass the date of the meeting you want. The `meeting_schedule_id` is required.
`GET /meetings/2018-03-12`

The Meeting returned from SHOW is **NOT** guaranteed to have an id. Meetings are not saved in the database until necessary. Because of that, the combination of the `meeting_schedule_id` and the `date` should be used.

The `original_date` that's returned is the original time the Meeting is supposed to exist, in case the time is adjusted for a particular meeting date.

## Attendance

> Example Response

```shell
{
  "attendance": {
    "id": 15,
    "client_uuid": null,
    "invited_by_id": 123,
    "user_id": 1803,
    "contact_id": null,
    "meeting_id": 1776,
    "attended_at": null,
    "deleted_at": null,
    "created_at": "2018-03-18 03:00:43 AM",
    "updated_at": "2018-03-18 03:00:43 AM"
  }
}
```

> Example Request

```shell
{
  "attendance": {
    "client_uuid": null,
    "user_id": 1803,     # * Either user_id OR contact_id is required- NOT both
    "contact_id": null,  # * Either user_id OR contact_id is required- NOT both
    "meeting_id": 1776,  # * Required on first create
  }
}
```

An Attendance must always be attached to 1 User or Contact. It may be attached to either one, but it cannot be attached to both.

The current user that invited the user/contact is referred to as the `invited_by` user.
