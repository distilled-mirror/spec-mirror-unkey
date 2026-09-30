> ## Documentation Index
> Fetch the complete documentation index at: https://unkey.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

> ## Agent Instructions
> Unkey is two separate products. Compute builds, deploys, and runs apps behind a gateway. API Management issues API keys, enforces rate limits, manages identities and permissions, and reports usage. Say which product a page belongs to; a reader can use either without the other.
> Every Unkey API endpoint is an HTTP POST to https://api.unkey.com/v2/{service}.{procedure} with a root key in the Authorization: Bearer header. Root keys are workspace scoped.
> Error codes have the form err:{system}:{category}:{specific} and each has a page at /errors/{system}/{category}/{specific}.
> The word environment means production or preview in Compute. Rate limiting has four meanings on this site; the glossary lists them.

# API authentication

> Send your root key as a bearer token and read the errors a bad key returns.

Every call to `https://api.unkey.com` needs a root key as a bearer token. The only exception is the customer portal's end-user endpoints, which take a portal session instead.

## Send a root key

Put the key in the `Authorization` header after `Bearer `:

```bash theme={"system"}
curl -X POST https://api.unkey.com/v2/apis.listKeys \
  -H "Authorization: Bearer unkey_1234abcd..." \
  -H "Content-Type: application/json" \
  -d '{"apiId": "api_1234abcd"}'
```

A root key belongs to one workspace, and every request stays inside it. You don't pass a workspace ID, and a key can't reach another workspace. See [Root keys](/docs/platform/root-keys/overview) to create one.

## Errors a bad key returns

| Status | Code | What it means |
| - | - | - |
| 400 | `err:unkey:authentication:missing` | There's no `Authorization` header. |
| 400 | `err:unkey:authentication:malformed` | The header is missing the `Bearer ` prefix, or has nothing after it. |
| 401 | `err:unkey:authentication:key_not_found` | The key doesn't exist or was deleted. |
| 403 | `err:unkey:authorization:key_disabled` | The key is disabled. |
| 403 | `err:unkey:authorization:workspace_disabled` | The key's workspace is disabled. |
| 403 | `err:unkey:authorization:forbidden` | The key has expired, for example because a rotation's grace period ended. |
| 403 | `err:unkey:authorization:insufficient_permissions` | The key doesn't have the permission the endpoint needs. The `detail` names it. Some endpoints return `404` instead, the same as if the resource didn't exist. See [Root key permissions](/docs/platform/root-keys/permissions). |

All of these use the standard [error envelope](/docs/platform/api/errors), so one error handler covers them.

## Keep root keys safe

Treat a root key like a database password:

* Create one per service with only the permissions that service needs. A leaked key then does less damage and is easy to replace.
* Never put a root key in client-side code, a mobile app, or a public repository.
* If one leaks, rotate or delete it under **Settings > Root Keys**. See [Root keys](/docs/platform/root-keys/overview).

Root keys are for managing Unkey. They aren't the API keys you issue to your own users, which you check with `keys.verifyKey`. See [Verifying keys](/docs/api-management/keys/verifying-keys).
