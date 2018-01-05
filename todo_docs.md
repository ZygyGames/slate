## Register

This endpoint authorizes a new user. All fields are required.

### HTTP Request

`GET https://maxactivity.com/api/v1/register`

`GET http://staging.maxactivity.com/api/v1/register`

> Example request

```shell
curl -X GET "https://maxactivity.com/api/v1/register"
curl -X GET "http://staging.maxactivity.com/api/v1/register"
  -H "App-Identifier: 123456789-987654321"
  -d "username=xxxxx&referral_code=xxxxxx&first_name=xxxxx&last_name=xxxxx&email=xxxxx@xxx.com&confirm_email=xxxxx@xxx.com&birthday=xx/xx/xx&password=xxxxxxxx&confirm_password=xxxxxxxx&accepted_tos=true"
```

### Request Expectations:

Type | Parameter | Required? | Description
---- | --------- | --------- | -----------
header | `App-Identifier` | YES | App Identification String
param | `username` | YES | Desired username. Must be between 2 and 20 characters
param | `referral_code` | YES | Zygy ID of the users friend that referred them to Zygy.
param | `first_name` | YES | User's First Name
param | `last_name` | YES | User's Last Name
param | `email` | YES | User's email address
param | `confirm_email` | YES | Confirmation email, must match `email` field.
param | `birthday` | YES | Birthday in the format MM/DD/YYYY
param | `password` | YES | Desired password. Must be 8 characters or more.
param | `confirm_password` | YES | Confirm password - Must match `password` field
param | `accepted_tos` | YES | Checkbox confirming user agrees to TOS and Privacy Policy

> Response - Failure

```shell
-H Authorization-Success : false
-H Error-Message : Username already taken.

{
  "errors": ["Username already taken."]
}
```

> Response - Success

```shell
status 200 - ok
-H Authorization-Success : true
-H Authorization-Code : OUKdeYQf1qiEoTF8clnk

{
  "username": "JohnDoe67",
  "server_id": "4",
  "zygy_id": "XUPR71"
}
```

### Response Expectations

Type | Key | Success? | Description
---- | --- | -------- | -----------
header | `Authorization-Success` | N/A | Stringified boolean representing whether or not the request was successful.
header | `Error-Message` | NO | Same message as json-error, passed as a Header.
json | `error` | NO | Description of why a failure resulted - Displayable to user
header | `Authorization-Code` | YES | Authorization Code for user. Save this within the app.
json | `username` | YES | Username of logged in user.
json | `server_id` | YES | Id for user on the server for URL requests only. (Uncommonly used)
json | `zygy_id` | YES |  6 digit Zygy ID of user. Commonly used and displayable

<aside class="success">
Remember — Store the `Authorization-Code` in-app and over-write it every time a user logs in!
</aside>

# Games

## Index

Returns a json array of every game.

Only display the games with `published: true` to the user.

### HTTP Request

`GET https://maxactivity.com/api/v1/games`

`GET http://staging.maxactivity.com/api/v1/games`

> Example request

```shell
curl -X GET "https://maxactivity.com/api/v1/games"
curl -X GET "http://staging.maxactivity.com/api/v1/games"
```

### Request Expectations

> Response

```shell

[
  {
    "server_id":"1",
    "name":"Zygy App",
    "updated_at":"2016-06-15T04:15:33.062Z",
    "how_to_description":null,
    "game_description":null,
    "app_store_id":null,
    "published":false,
    "app_icon":"/system/games/app_icons/missing.png"
  },
  {
    "server_id":"2",
    "name":"Stack Em",
    "updated_at":"2016-06-15T04:21:35.661Z",
    "how_to_description":"",
    "game_description":"Build them as high as you can.",
    "app_store_id":null,
    "published":false,
    "app_icon":"http://s3.amazonaws.com/zygy-user-image-uploads/games/app_icons/000/000/004/original/Stack-Em_Icon-20.png?1465950356"
  },
  {
    "server_id":"3",
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

### Response Expectations

Type | Key  | Description
---- | ---  | -----------
json | `server_id` | The server ID of the game used for requests for a specific game.
json | `name` | Name of the game
json | `updated_at` | The last time the game was updated
json | `game_description` | Description of what the game is
json | `how_to_description` | How to play the game
json | `app_store_id` | If the game is available on the App Store, the ID used to identify it.
json | `published` | Whether or not the game is published and should be shown to the user.
json | `app_icon` | The url to the app's icon

## Check Rewards

Returns a boolean on whether or not the current user has any pending rewards for a the current game.

Current game is determined using App-Identifier

### HTTP Request

`GET https://maxactivity.com/api/v1/games/check_rewards`

`GET http://staging.maxactivity.com/api/v1/games/check_rewards`

> Example request

```shell
curl -X GET "https://maxactivity.com/api/v1/games/check_rewards"
curl -X GET "http://staging.maxactivity.com/api/v1/games/check_rewards"
```

### Request Expectations

> Response

```shell
true
```

### Response Expectations

## Collect Rewards

Returns a hash of data including all of the rewards to give to a player for the current game.

### HTTP Request

`GET https://maxactivity.com/api/v1/games/collect_rewards`

`GET http://staging.maxactivity.com/api/v1/games/collect_rewards`

> Example request

```shell
curl -X GET "https://maxactivity.com/api/v1/games/collect_rewards"
curl -X GET "http://staging.maxactivity.com/api/v1/games/collect_rewards"
```

### Request Expectations

> Response

```shell
{
  "coins": 100,
  "gems": 15
}
```

### Response Expectations

Keys are mapped to the string value of the type of reward to receive. (Usually coins)

Values are mapped to the amount of that reward to increment.

# Zygy Now

Zygy Now is a a Scoreboard based tracker that allows a User to track their information. It contains information for the selected month, as well as an 'all-time best'.

A user can select which month to view, and can also change the user in which information is shown. A user can ONLY view a different user's information if the perspective user is a 'downline' of the current user.

All values and dates sent as a response from the server will be pre-formatted for display.

### HTTP Request

`GET https://maxactivity.com/api/v1/zygy_now`

`GET http://staging.maxactivity.com/api/v1/zygy_now`

### Request Expectations

> Example request

```shell
# params are optional
curl -X GET "https://maxactivity.com/api/v1/zygy_now"
curl -X GET "http://staging.maxactivity.com/api/v1/zygy_now"
  -H "App-Identifier: 123456789-987654321"
  -H "Authorization-Code: OUKdeYQf1qiEoTF8clnk"
  -d "user_identification=rockster160&month=4&year=2016"
```

Type | Parameter | Required? | Description
---- | --------- | --------- | -----------
header | `App-Identifier` | YES | App Identification String
header | `Authorization-Code` | YES | User Authorization Code
params | `user_identification` | NO | `Default: current user` If supplied, looks up user by this value. (Username, Email, or Zygy ID)
params | `month` | NO | `Default: current month` If supplied, filters dates based on this integer value. (1=January, 12=December)
params | `year` | NO | `Default: current year` If supplied, filters dates based on this integer value. (YYYY)

### Response Expectations

> Response - Failure

```shell
-H Authorization-Success : false
-H Error-Message : Cannot view a user unless they are on your team.

{
  "errors": ["Cannot view a user unless they are on your team."]
}
```

> Response - Success

```shell
status 200 - ok
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
header | `Error-Message` | NO | Same message as json-error, passed as a Header.
json | `error` | NO | Description of why a failure resulted - Displayable to user
json | `current_month` | YES | The currently scoped month as a String for display
json | `data` | YES | Array of 'data rows' containing values for each section.
json | `name` | YES | The name of either the game or the type of data the row contains
json | `current_month_value` | YES | String value of row at selected month
json | `best_month` | YES | String value of date the `best_month_value` occurred.
json | `best_month_value` | YES | String value of row at best month

# News Feed

News Feed is a list of articles that contain basic information shown to users.

The News Feed will return a Webview, and should be rendered as is, using the standard App's Navigation.

That means showing a back button and the normal tab view that is displayed on other pages.

## Endpoint

`GET https://maxactivity.com/api/v1/news_feed`

`GET http://staging.maxactivity.com/api/v1/news_feed`

### Request Expectations

> Example request

```shell
# params are optional
curl -X GET "https://maxactivity.com/api/v1/news_feed"
curl -X GET "http://staging.maxactivity.com/api/v1/news_feed"
  -H "App-Identifier: 123456789-987654321"
  -H "Authorization-Code: OUKdeYQf1qiEoTF8clnk"
  -d "page=4&per=2"
```

Type | Parameter | Required? | Description
---- | --------- | --------- | -----------
header | `App-Identifier` | YES | App Identification String
header | `Authorization-Code` | YES | User Authorization Code

### Response Expectations

> Response - Failure

```shell
-H Authorization-Success : false
-H Error-Message : Authorization failure.

{
  "errors": ["Authorization failure."]
}
```

> Response - Success

```shell
status 200 - ok
<!DOCTYPE html>
<html>
 <!-- The News feed with associated styles should be here. -->
</html>
```

# Leaderboard

The Leaderboard endpoint will return a JSON response of the current Leaderboards based on your request.

Leaderboards are slow to calculate, but to make up for this, once a Leaderboard is viewed, it is cached and stored in the server so future lookups are much faster. If you notice an issue with a slow response from a Leaderboard, try again and it should respond very quickly since it doesn\'t have to regenerate the information.

The Leaderboard endpoint is currently unauthenticated, so you do not have to send the App Identifier and Authorization Code with your request. (Although it is recommended, when possible.)

## Rankings for Game

Returns an array of users with their current rankings in the selected Leaderboard

### HTTP Request

`GET https://maxactivity.com/api/v1/leaderboards/games/:game_server_id`

`GET http://staging.maxactivity.com/api/v1/leaderboards/games/:game_server_id`

> Example request

```shell
curl -X GET "https://maxactivity.com/api/v1/leaderboards/games/2"
curl -X GET "http://staging.maxactivity.com/api/v1/leaderboards/games/2"
  -d "depth=thru&base_level=Personal&tracker_type=Scores&time_range=Current+Month&filter_since=before&filter_user_date=01-17-2017"
```

### Request Expectations

Type | Parameter | Required? | Description
---- | --------- | --------- | -----------
param | `game_server_id` | YES | <This value should be passed inside the URL> The server ID of the game you want the Leaderboards for. The ID is accessible via the `games` endpoint.
param | `depth` | NO | Thru/At - Determines whether the Leaderboards should be filtered 'thru' the current level or 'at' the current level. <Default: thru>
param | `base_level` | NO | Ordinalized depth value. May be a string or integer value Personal/0 for Personal, 1st/1, 2nd/2, ... Up to 8th/8 and total. Any number greater than 20 may be passed for total.
param | `tracker_type` | NO | "Scores", "Revenue", "New Players Added" - for the type of Leaderboard to show.
param | `time_range` | NO | String description of relative date range to include in the Leaderboard. Valid options: "Current Month", "Last Month", "Last 3 Months", "Last 6 Months", "Last 12 Months", "Year To Date", "Total"
param | `filter_since` | NO | Boolean value, true marks to filter users created after the `filter_user_date` below, false will show only users created before that date.
param | `filter_user_date` | NO | Date in the form of MM-DD-YYYY, when provided, will filter the Leaderboard to only include users based on this date.

> Response - Failure

```shell

{
  "errors": [{]
    "required_params_not_present": [
      "game"
    ]
  }
}
```

> Response - Success

```shell
status 200 - ok
[
  {
    "place":1,
    "username_display":"Chad510 (GP9ITD)",
    "personal_value":"1,939",
    "filtered_value":"1,939",
    "upline_display":"Carmella453 (ZV3P7T)"
  },
  {
    "place":2,
    "username_display":"Jaron77 (8Y106M)",
    "personal_value":"1,893",
    "filtered_value":"1,893",
    "upline_display":"Jess52 (62XFC1)"
  },
  ...
]
```

### Response Expectations

Type | Key | Success? | Description
---- | --- | -------- | -----------
json | `place` | YES | Rank on the current Leaderboard
json | `username_display` | YES | String value including the User\'s Username followed by their Zygy ID in parentheses.
json | `personal_value` | YES | The value representing the individual score for the User (Will show the Current User\'s high score whether or not the filter is set to `thru` any level)
json | `filtered_value` | YES | The value the current Leaderboards are sorted by.
json | `upline_display` | YES | String value displaying the User\'s upline\'s identification.

## Rankings for User

Returns an array of users with their current rankings in the selected Leaderboard

If no parameters are passed in, will return only the selected user.

### HTTP Request

`GET https://maxactivity.com/api/v1/leaderboards/games/:game_server_id/users/:user_server_id`

`GET http://staging.maxactivity.com/api/v1/leaderboards/games/:game_server_id/users/:user_server_id`

> Example request

```shell
curl -X GET "https://maxactivity.com/api/v1/leaderboards/games/2/users/174"
curl -X GET "http://staging.maxactivity.com/api/v1/leaderboards/games/2/users/174"
  -d "padding=3"
```

### Request Expectations

Type | Parameter | Required? | Description
---- | --------- | --------- | -----------
param | `game_server_id` | YES | <This value should be passed inside the URL> The server ID of the game you want the Leaderboards for. The ID is accessible via the `games` endpoint.
param | `user_server_id` | YES | <This value should be passed inside the URL> The server ID of the User you want the rankings for. This ID is returned in the authentication request.
param | `padding` | NO | Integer - Sets the values of `padding_top` and `padding_bottom` both to this value.
param | `padding_top` | NO | Integer - When passed, includes the given number of users ranking HIGHER than the selected user.
param | `padding_bottom` | NO | Integer - When passed, includes the given number of users ranking LOWER than the selected user.

> Response - Failure

```shell
{
  "errors": ["Authorization Failure"]
}
```

> Response - Success

```shell
status 200 - ok
# No padding:
[
  {
    "place": 359,
    "username_display": "Pearlie174 (XZRC8I)",
    "personal_value": "1,423",
    "filtered_value": "1,423",
    "upline_display": "Sandra162 (YW7GQR)"
  }
]

# Padding = 2 (top: 2, bottom: 2)
[
  {
    "place": 356,
    "username_display": "Rogelio775 (Z6PKYW)",
    "personal_value": "1,425",
    "filtered_value": "1,425",
    "upline_display": "Lelia531 (VRF375)"
  },
  {
    "place": 357,
    "username_display": "Rachel730 (95P8RW)",
    "personal_value": "1,424",
    "filtered_value": "1,424",
    "upline_display": "Linnie643 (0ALK6Q)"
  },
  {
    "place": 359,
    "username_display": "Pearlie174 (XZRC8I)",
    "personal_value": "1,423",
    "filtered_value": "1,423",
    "upline_display": "Sandra162 (YW7GQR)"
  },
  {
    "place": 359,
    "username_display": "Milford183 (EWD934)",
    "personal_value": "1,422",
    "filtered_value": "1,422",
    "upline_display": "Bud43 (4DQGI1)"
  },
  {
    "place": 360,
    "username_display": "Eloy391 (U90DAK)",
    "personal_value": "1,421",
    "filtered_value": "1,421",
    "upline_display": "Brennan251 (3F60NM)"
  }
]
```

### Response Expectations

Type | Key | Success? | Description
---- | --- | -------- | -----------
json | `place` | YES | Rank on the current Leaderboard
json | `username_display` | YES | String value including the User\'s Username followed by their Zygy ID in parentheses.
json | `personal_value` | YES | The value representing the individual score for the User (Will show the Current User\'s high score whether or not the filter is set to `thru` any level)
json | `filtered_value` | YES | The value the current Leaderboards are sorted by.
json | `upline_display` | YES | String value displaying the User\'s upline\'s identification.

## Golf Scores Leaderboard

Returns an array of users with their current rankings in golf scores

### HTTP Request

`GET https://maxactivity.com/api/v1/leaderboards/golf`

`GET http://staging.maxactivity.com/api/v1/leaderboards/golf`

> Example request

```shell
curl -X GET "https://maxactivity.com/api/v1/leaderboards/golf"
curl -X GET "http://staging.maxactivity.com/api/v1/leaderboards/golf"
  -d "depth=thru&base_level=Personal&tracker_type=Scores&time_range=Current+Month&filter_since=before&filter_user_date=01-17-2017"
```

### Request Expectations

Type | Parameter | Required? | Description
---- | --------- | --------- | -----------
param | `depth` | NO | Thru/At - Determines whether the Leaderboards should be filtered 'thru' the current level or 'at' the current level. <Default: thru>
param | `base_level` | NO | Ordinalized depth value. May be a string or integer value Personal/0 for Personal, 1st/1, 2nd/2, ... Up to 8th/8 and total. Any number greater than 20 may be passed for total.
param | `tracker_type` | NO | "Scores", "Revenue", "New Players Added" - for the type of Leaderboard to show.
param | `time_range` | NO | String description of relative date range to include in the Leaderboard. Valid options: "Current Month", "Last Month", "Last 3 Months", "Last 6 Months", "Last 12 Months", "Year To Date", "Total"
param | `filter_since` | NO | Boolean value, true marks to filter users created after the `filter_user_date` below, false will show only users created before that date.
param | `filter_user_date` | NO | Date in the form of MM-DD-YYYY, when provided, will filter the Leaderboard to only include users based on this date.

> Response - Success

```shell
status 200 - ok
[
  {
    "place":1,
    "username_display":"Chad510 (GP9ITD)",
    "personal_value":"6",
    "filtered_value":"6",
    "upline_display":"Carmella453 (ZV3P7T)"
  },
  {
    "place":2,
    "username_display":"Jaron77 (8Y106M)",
    "personal_value":"9",
    "filtered_value":"9",
    "upline_display":"Jess52 (62XFC1)"
  },
  ...
]
```

### Response Expectations

Type | Key | Success? | Description
---- | --- | -------- | -----------
json | `place` | YES | Rank on the current Leaderboard
json | `username_display` | YES | String value including the User\'s Username followed by their Zygy ID in parentheses.
json | `personal_value` | YES | The value representing the individual score for the User (Will show the Current User\'s high score whether or not the filter is set to `thru` any level)
json | `filtered_value` | YES | The value the current Leaderboards are sorted by.
json | `upline_display` | YES | String value displaying the User\'s upline\'s identification.

## Golf Rankings for User

Returns an array of users with their current rankings in the selected Leaderboard

If no parameters are passed in, will return only the selected user.

### HTTP Request

`GET https://maxactivity.com/api/v1/leaderboards/golf/users/:user_server_id`

`GET http://staging.maxactivity.com/api/v1/leaderboards/golf/users/:user_server_id`

> Example request

```shell
curl -X GET "https://maxactivity.com/api/v1/leaderboards/golf/users/174"
curl -X GET "http://staging.maxactivity.com/api/v1/leaderboards/golf/users/174"
  -d "padding=3"
```

### Request Expectations

Type | Parameter | Required? | Description
---- | --------- | --------- | -----------
param | `user_server_id` | YES | <This value should be passed inside the URL> The server ID of the User you want the rankings for. This ID is returned in the authentication request.
param | `padding` | NO | Integer - Sets the values of `padding_top` and `padding_bottom` both to this value.
param | `padding_top` | NO | Integer - When passed, includes the given number of users ranking HIGHER than the selected user.
param | `padding_bottom` | NO | Integer - When passed, includes the given number of users ranking LOWER than the selected user.

> Response - Failure

```shell
{
  "errors": ["Authorization Failure"]
}
```

> Response - Success

```shell
status 200 - ok
# No padding:
[
  {
    "place": 359,
    "username_display": "Pearlie174 (XZRC8I)",
    "personal_value": "456",
    "filtered_value": "456",
    "upline_display": "Sandra162 (YW7GQR)"
  }
]

# Padding = 2 (top: 2, bottom: 2)
[
  {
    "place": 356,
    "username_display": "Rogelio775 (Z6PKYW)",
    "personal_value": "403",
    "filtered_value": "403",
    "upline_display": "Lelia531 (VRF375)"
  },
  {
    "place": 357,
    "username_display": "Rachel730 (95P8RW)",
    "personal_value": "437",
    "filtered_value": "437",
    "upline_display": "Linnie643 (0ALK6Q)"
  },
  {
    "place": 359,
    "username_display": "Pearlie174 (XZRC8I)",
    "personal_value": "456",
    "filtered_value": "456",
    "upline_display": "Sandra162 (YW7GQR)"
  },
  {
    "place": 359,
    "username_display": "Milford183 (EWD934)",
    "personal_value": "480",
    "filtered_value": "480",
    "upline_display": "Bud43 (4DQGI1)"
  },
  {
    "place": 360,
    "username_display": "Eloy391 (U90DAK)",
    "personal_value": "502",
    "filtered_value": "502",
    "upline_display": "Brennan251 (3F60NM)"
  }
]
```

### Response Expectations

Type | Key | Success? | Description
---- | --- | -------- | -----------
json | `place` | YES | Rank on the current Leaderboard
json | `username_display` | YES | String value including the User\'s Username followed by their Zygy ID in parentheses.
json | `personal_value` | YES | The value representing the individual score for the User (Will show the Current User\'s high score whether or not the filter is set to `thru` any level)
json | `filtered_value` | YES | The value the current Leaderboards are sorted by.
json | `upline_display` | YES | String value displaying the User\'s upline\'s identification.
