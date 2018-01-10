# Main Title

## Endpoint

This endpoint does some stuff. Use it wisely

### HTTP Request

`GET https://maxactivity.com/api/v1/...`

This request does some stuff.

> Example request

```shell
curl -X GET "https://maxactivity.com/api/v1/..."
  -H "Authorization: Token aaabbbcccdddeeefffggghhhiii"
  -d "param=val"
```

### Request Expectations:

Type | Parameter | Required? | Description
---- | --------- | --------- | -----------
header | `Authorization` | YES | `string`
param | `param` | NO | Array of specific user ids to request.

> Response - Success

```shell
status 200 - OK
-H Authorization Token: aaabbbcccdddeeefffggghhhiii

{
  "username": "JohnDoe67",
  "server_id": "4",
  "zygy_id": "XUPR71"
}
```

### Response Expectations

Type | Key | Success? | Description
---- | --- | -------- | -----------
json | `errors` | NO | `hash:array<string>`
header | `Authorization Token` | YES | `string`

<aside class="warning">
Remember — Store the `Authorization-Code` in-app and over-write it every time a user logs in!
</aside>
