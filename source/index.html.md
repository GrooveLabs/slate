---
title: Groove API Reference

language_tabs:
  - cURL

toc_footers:
  - <a href="mailto:support@groove.com"'>Become an API partner</a>

search: true

---

# Introduction

Welcome to the Groove API!

Hopefully, this documentation can help you get up and running as quickly as possible.

Feel free to reach out to <a href="mailto:support@groove.com">our support team</a> if you have any additional questions.

# Authentication

Right now, we support the ability for registered API partners to access our API on behalf of Groove users via an access token. If you are interested in becoming a registered API partner, please <a href="mailto:support@groove.com">contact us</a>.

## OAuth Flow

We use [the OAuth 2.0 protocol](https://tools.ietf.org/html/rfc6749) to generate access tokens.

### Step 1: Redirect Users

To initiate the authorization process, redirect users to `https://auth.grooveapp.com/oauth/authorize`.

Provide the following query parameters

| key           | value                                                                                        |
|---------------|----------------------------------------------------------------------------------------------|
| client_id     | the client ID configured for your OAuth App                                                  |
| redirect_uri  | the redirect URI configured for your OAuth App                                               |
| scope         | space-separated list of scopes (see relevant section below)                                  |
| state         | an optional string to be passed back upon completion                                         |

### Step 2: Handle User Authorization

If the user authorizes your application, Groove will redirect to the specified `redirect_uri`.

This redirection will contain a temporary `code` query parameter. These authorization `code`s may only be exchanged once, and will expire after 10 minutes.

This redirection will also contain the previously passed `state` value, as a query parameter. If this `state` value does not match, the request may be compromised and you should abort the authorization

### Step 3: Exchange the authorization code for an access token

Make a `POST` request to `https://auth.grooveapp.com/oauth/token`

```shell
curl -X POST "https://auth.grooveapp.com/oauth/token"
  -H "Content-Type: application/x-www-form-urlencoded"
  -d "grant_type=authorization_code&code=SomeAuthorizationCode&redirect_uri=SomeRedirectUri&client_id=SomeClientId&client_secret=SomeClientSecret"
```

> The returned `JSON` will look something like

```json
{
  "access_token": "some-access-token",
  "created_at": "685238400",
  "token_type": "Bearer",
  "expires_in": 7200,
  "refresh_token": "some-refresh-token",
  "scope": "first-scope second-scope",
}
```

The request body should include the following values:

| key           | value                                                                                        |
|---------------|----------------------------------------------------------------------------------------------|
| grant_type    | `authorization_code`                                                                         |
| code          | the code you were provided in the previous step                                              |
| redirect_uri  | the redirect URI configured for your OAuth App                                               |
| client_id     | the client ID configured for your OAuth App                                                  |
| client_secret | the client secret configured for your OAuth App                                              |

**Note: Please avoid exposing your client secret**

The returned response body should contain a `JSON` payload with the following key/value pairs

| key           | value                                                                                        |
|---------------|----------------------------------------------------------------------------------------------|
| access_token  | the value represents the string to add to the `Authorization`  header of future API requests |
| created_at    | the value represents the Unix timestamp when the access token was created                    |
| token_type    | the value will be `Bearer`                                                                   |
| expires_in    | the value indicates the number of seconds until the access token will be invalid             |
| refresh_token | the value indicates the string to use to generate another access token                       |
| scope         | a space-separated string indicating the scopes that the access token can use                 |

## Making Authenticated API Calls

Use the generated `access_token` value to make API requests on behalf of the authorized user in the `Authorization` header with the header value having the following format: `Authorization: Bearer {YOUR_ACCESS_TOKEN}`

> An authenticated CURL may look like:

```
curl -X POST "https://app.grooveapp.com/api/public/..."
  -H 'Content-Type: application/json'
  -H 'Authorization: Bearer {YOUR_ACCESS_TOKEN}'
  -d ...
```


## Access Token Expiration

An access token expires 2 hours after creation. The expiration time is specified by the `expires_in` field in the token creation response body.

In order to generate a new access token, use the refresh token, specified by the `refresh_token` field in the token creation response body, to make a `POST` request to `/oauth/token` with the refresh token.


| key           | value                                                                                        |
|---------------|----------------------------------------------------------------------------------------------|
| grant_type    | `refresh_token`                                                                         |
| refresh_token | YOUR_REFRESH_TOKEN
| redirect_uri  | the redirect URI configured for your OAuth App                                               |
| client_id     | the client ID configured for your OAuth App                                                  |
| client_secret | the client secret configured for your OAuth App                                              |

```shell
curl -X POST "https://auth.grooveapp.com/oauth/token"
  -H 'Content-Type: application/json'
  -d '{
    "grant_type": "refresh_token",
    "refresh_token": "SomeRefreshToken",
    "redirect_url": "SomeRedirectUri",
    "client_id": "SomeClientId",
    "client_secret": "SomeClientSecret"
  }'
```

# Rate Limiting

In general, we do not allow more than `100` API requests, per minute, per OAuth token. For every request made, these response headers will be returned:

Header | Description
--------- | -----------
`X-RateLimit-Limit` | The maximum number of API calls able to be made in the current minute
`X-RateLimit-Remaining` | The number of API calls left in the current minute
`X-RateLimit-Reset` | The time (in epoch seconds) at which API calls are guaranteed to be available again

We perform a sliding window calculation across the current and last minutes to prevent call bursts around the minute boundary. If a limit is hit, the reset value is calculated to be the moment in time when the sliding average dips below the limit.

Exceeding the limit will yield an empty response with status code `429`.

**Note: We continue to count any API calls made while the limit is exceeded.**


# Scoping

Scopes are defined at an OAuth App level. The below table describes the scopes and the corresponding GraphQL queries/mutation that they allow.

| Scope                | Description                                    | Queries                                      | Mutations                  |
|----------------------|------------------------------------------------|----------------------------------------------|----------------------------|
| `user::flow:read`    | View your flows, flow steps, and flow people   | `flow`, `flows`, `findAddableFlowsForPerson` |                            |
| `user::flow:write`   | Modify your flows, flow steps, and flow people |                                              | `addPersonToFlow`          |
| `user::people:read`  | View your people                               |                                              | `findPersonBySalesforceId` |
| `user::people:write` | Modify your people                             |                                              | `createPerson`             |


When a resource owner is authenticating your OAuth App, they will see a screen like this:

![](consent_dialog.png)

# API

Our GraphQL API only has one endpoint - `https://app.grooveapp.com/api/public/v1/graphql`.

One of the benefits of GraphQL is that allows the client to specify _exactly_ what fields they would like as part of the response payload.

Another benefit of GraphQL is it's statically-generated schema that allows introspection into the various query and mutation operations available.

You can checkout our GraphQL documentation here.

Again, the GraphQL schema, and the above documentation, will specify the exact objects and their respective fields for queries and mutations. The following example API responses _are not_ an exhaustive representation.

