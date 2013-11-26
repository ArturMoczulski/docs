# Invites

<!-- Sub:[TOC] -->

## Get an invite

    GET /api/1/invite/:id


## List outstanding invites for a team

    GET /api/1/team/:id/invites


## Invite a user to a team

    POST /api/1/team/:id/invites

### Parameters

Name | Type | Description
-----|------|-------------
`email`|`string`|**Required.** Email address to invite.

Params must be supplied as JSON, and as the body of the request. Be sure to set the header `Content-Type: application/json`.


## Cancel an invite

Cancels an invite, making it no longer valid to accept. If it has already been accepted, use `DELETE /team/:team_id/user/:user_id` instead.

    DELETE /api/1/invite/:id


-----

Last updated: November 26, 2013
