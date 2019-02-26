# What is Enceeper?

[Enceeper](https://www.enceeper.com/) is a service used to store and organize users’ passwords and secrets. It provides high level of security since users’ information is not directly stored to the service.

The [Enceeper App](https://github.com/enceeper/enceeper) is using the service in the following way:

- All data are first encrypted locally and then transmitted over the network.
- The password of the user is never transmitted, but is used locally to compute a proof-of-knowledge.
- Moreover, a user has the option to "share" those secrets with other users of the service by adding slots to selected entries.
- Finally, any third party can use the API of the service to enhance the core functionality.

To this end we have developed two solutions:

- [enceeper-phpconf](https://github.com/enceeper/enceeper-phpconf) to store configuration information in Enceeper and have it delivered to a PHP application
- [enceeper-boot](https://github.com/enceeper/enceeper-boot) for unattended booting a GNU/Linux distro via Initramfs utilizing Enceeper

# API documentation

In this document we describe the available API calls, the input they receive and the output they produce.

The current API version is: **1.0.0** and the base URL of the service is: [https://www.enceeper.com/api/v1](https://www.enceeper.com/api/v1)

The API uses [HTTP response codes](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes) to indicate success or failure of requests. Specifically, codes in the 2xx range indicate success, codes in the 4xx range indicate an error that resulted from the provided arguments (e.g. instead of an integer a string is provided) and 5xx error codes indicate an internal Enceeper error.

> There is only one error that can be handled in a special way, the 403 error. This error is generated when the auth token has expired and the user needs to re-authenticate. The client can then check for this error and re-authenticate the user without requiring any interaction.

All inputs and outputs follow the JSON format and based on whether the outcome of an API call was sucessfull or not we have the following response templates:

Sucess response:
```
{
  "outcome": "OK",
  "result": {
    "data": {
      "one": "...",
      "two": "..."
    }
  },
  "errorMessage": null
}
```

> If an API call does not return any data then the **result** key above will be null.

Failed response:
```
{
  "outcome":      "NOK",
  "result":       null,
  "errorMessage": "API method not found"
}
```

## API calls

The summary of the API calls is:

| Title                                               | Method | URL                               |
|-----------------------------------------------------|--------|-----------------------------------|
| [Test call](#test-call)                                    | GET    | /                                 |
| [User registration](user.md#user-registration)             | POST   | /user                             |
| [Initiate auth procedure](user.md#initiate-auth-procedure) | POST   | /user/challenge                   |
| [Authenticate user](user.md#authenticate-user)             | POST   | /user/login                       |
| [Get specific key](#get-specific-key)               | GET    | /user/slots/{identifier}          |
| [Check for key approval](#check-for-key-approval)   | GET    | /user/slots/check/{ref}           |
| [Edit user](#edit-user)*                            | PUT    | /user                             |
| [Delete user](#delete-user)*                        | DELETE | /user                             |
| [Get account keys](#get-account-keys)*              | GET    | /user/keys                        |
| [Create new key](#create-new-key)*                  | POST   | /user/keys                        |
| [Edit key](#edit-key)*                              | PUT    | /user/keys/{keyId}                |
| [Delete key](#delete-key)*                          | DELETE | /user/keys/{keyId}                |
| [Create new slot](#create-new-slot)*                | POST   | /user/keys/{keyId}/slots          |
| [Edit slot](#edit-slot)*                            | PUT    | /user/keys/{keyId}/slots/{slotId} |
| [Delete slot](#delete-slot)*                        | DELETE | /user/keys/{keyId}/slots/{slotId} |
| [Search users](#search-users)*                      | POST   | /user/search                      |
| [Create key share](#create-key-share)*              | POST   | /user/keys/{keyId}/share          |
| [Accept key share](#accept-key-share)*              | POST   | /user/keys/shares/{shareId}       |
| [Delete key share](#delete-key-share)*              | DELETE | /user/keys/shares/{shareId}       |

> All API calls marked with * require the use of the **X-Enceeper-Auth** HTTP header with a valid authentication token retrieved from the **Authenticate user** API call.

### Test call

Test API call to verify network connectivity and check that the Enceeper service is working as expected.

| Type   | Value|
|--------|-|
| URL    | /|
| Method | GET|
| Input  | -|
| Output | -|

### Get specific key

Get the key details of the provided **identifier** (for details see **Get account keys**). The owner of the slot can choose one of the following settings:

- Give the slot immediately to the caller
- Give the slot to the caller, but notify me by email that this slot was used
- Do not provide the slot and wait for manual approval by the owner

So the response can be one of the following:

- For the first two scenarios the **slot**, **meta** and **value** keys are populated
- For the last scenario the **ref** and **ttl** keys are populated

And the user must call the **Check for key approval** with the provided **ref** in order to get the key details. The **ttl** is the number of seconds the **ref** will be valid, before it expires.

| Type   | Value|
|--------|-|
| URL    | /user/slots/{identifier}|
| Method | GET|
| Input  | -|
| Output | {<br>&nbsp;"ref": "string ref",<br>&nbsp;"ttl": time in seconds that the ref expires,<br>&nbsp;"slot": "string encrypted slot",<br>&nbsp;"meta": "string encrypted meta",<br>&nbsp;"value": "string encrypted value"<br>}|

### Check for key approval

Check if the provided **ref** has been approved or not. If it has not been approved an error is returned (HTTP status code 428). If the request was approved by the slot owner the **slot**, **meta** and **value** keys are populated. For details on the **ref** parameter read the **Get specific key** API call above.

| Type   | Value|
|--------|-|
| URL    | /user/slots/check/{ref}|
| Method | GET|
| Input  | -|
| Output | {<br>&nbsp;"slot": "string encrypted slot",<br>&nbsp;"meta": "string encrypted meta",<br>&nbsp;"value": "string encrypted value"<br>}|

### Edit user

Update user details. For the **auth** object the same constrains are in place as described above in the **User registration** section.

| Type   | Value|
|--------|-|
| URL    | /user|
| Method | PUT|
| Input  | {<br>&nbsp;"auth": {<br>&nbsp;&nbsp;"srp6a": {<br>&nbsp;&nbsp;&nbsp;"salt": "hex salt",<br>&nbsp;&nbsp;&nbsp;"verifier": "hex verifier"<br>&nbsp;&nbsp;},<br>&nbsp;&nbsp;...<br>&nbsp;&nbsp;"keys": {<br>&nbsp;&nbsp;&nbsp;"pub": "the public key of the user used in key sharing",<br>&nbsp;&nbsp;&nbsp;...<br>&nbsp;&nbsp;},<br>&nbsp;&nbsp;...<br>&nbsp;}<br>}|
| Output | -|

### Delete user

| Type   | Value|
|--------|-|
| URL    | /user|
| Method | DELETE|
| Input  | -|
| Output | -|

### Get account keys

### Create new key

### Edit key

### Delete key

| Type   | Value|
|--------|-|
| URL    | /user/keys/{keyId}|
| Method | DELETE|
| Input  | -|
| Output | -|

### Create new slot

### Edit slot

### Delete slot

| Type   | Value|
|--------|-|
| URL    | /user/keys/{keyId}/slots/{slotId}|
| Method | DELETE|
| Input  | -|
| Output | -|

### Search users

Search for other users in the platform and retrieve their public key for key sharing.

| Type   | Value|
|--------|--------------------------------------------------------------------------------------|
| URL    | /user/search|
| Method | POST|
| Input  | {<br>&nbsp;"email": "enceeper@example.com"<br>}|
| Output | {<br>&nbsp;"sharePubKey": "hex pub key"<br>}|

### Create key share

### Accept key share

### Delete key share

Deletes a request to share a slot.

| Type   | Value|
|--------|-|
| URL    | /user/keys/shares/{shareId}|
| Method | DELETE|
| Input  | -|
| Output | -|

## Rate limiting

50 requests per hour (unauthenticated user)
XX requests per hour based on user plan

For details on the available plans visit: www.enceeper.com

X-RateLimit-Limit     100
X-RateLimit-Remaining 99
X-RateLimit-Reset     1551173510
