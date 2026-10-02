> ## Documentation Index
> Fetch the complete documentation index at: https://unkey.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

> ## Agent Instructions
> Unkey is two separate products. Compute builds, deploys, and runs apps behind a gateway. API Management issues API keys, enforces rate limits, manages identities and permissions, and reports usage. Say which product a page belongs to; a reader can use either without the other.
> Every Unkey API endpoint is an HTTP POST to https://api.unkey.com/v2/{service}.{procedure} with a root key in the Authorization: Bearer header. Root keys are workspace scoped.
> Error codes have the form err:{system}:{category}:{specific} and each has a page at /errors/{system}/{category}/{specific}.
> The word environment means production or preview in Compute. Rate limiting has four meanings on this site; the glossary lists them.

# HTTP API basics

> Call the Unkey API: one base URL, POST endpoints, and a shared response envelope.

Compute and API Management share one HTTP API. Here's how to make a request, read the response, and page through lists.

## Make a request

Send a `POST` with a JSON body to `https://api.unkey.com/v2/{service}.{procedure}`. There are no path or query parameters. The only `GET` routes are `/v2/liveness`, the health check, and `/openapi.yaml` and `/reference`, which serve the OpenAPI document and a browsable reference. See [RPC naming convention](/docs/platform/api/rpc-convention) for how endpoints are named.

```bash theme={"system"}
curl -X POST https://api.unkey.com/v2/keys.verifyKey \
  -H "Authorization: Bearer <root key>" \
  -H "Content-Type: application/json" \
  -d '{"key": "sk_live_..."}'
```

Put a root key in the `Authorization: Bearer` header. See [API authentication](/docs/platform/api/authentication).

## Read the response

Every response is a JSON object with `meta`. A success adds `data`, and a failure adds `error` instead.

<ResponseField name="meta" type="object" required>
  Present on every response.

  <Expandable title="properties">
    <ResponseField name="requestId" type="string" required>
      A unique ID for this request, prefixed `req_`. It's the only field in `meta`, and it isn't sent as a response header. Log it with every call and include it when you contact support, so we can find the exact request.
    </ResponseField>
  </Expandable>
</ResponseField>

<ResponseField name="data" type="object | array">
  The endpoint's result. Its shape is documented per endpoint in each product's API reference. List endpoints return an array here.
</ResponseField>

<ResponseField name="error" type="object">
  Present instead of `data` when the request failed. Its fields are on [API error envelope](/docs/platform/api/errors).
</ResponseField>

<ResponseField name="pagination" type="object">
  Present only on list endpoints. See [Page through lists](#page-through-lists).
</ResponseField>

```json A successful response theme={"system"}
{
  "meta": { "requestId": "req_2c9a0jf23l4k567" },
  "data": { "keyId": "key_1234abcd" }
}
```

## Page through lists

List endpoints use a cursor instead of page numbers. Each request takes an optional `limit` and `cursor`, and each response has a `pagination` object next to `data`. The example uses `apis.listKeys`, but every list endpoint works the same way.

<ParamField body="limit" type="integer">
  Maximum items to return. For `apis.listKeys` it's 1 to 100, default 100. Most list endpoints default to 100, but `ratelimit.listOverrides` defaults to 50. Send `limit` if the page size matters to you.
</ParamField>

<ParamField body="cursor" type="string">
  The `pagination.cursor` value from the previous response, sent back unchanged. Leave it out for the first page. Cursors can expire, so don't store them long term or build them yourself.
</ParamField>

<ResponseField name="pagination.hasMore" type="boolean" required>
  `true` when there's another page. When `false`, you have everything and `cursor` may be missing or `null`.
</ResponseField>

<ResponseField name="pagination.cursor" type="string">
  The value to send as `cursor` on the next request. Only use it while `hasMore` is `true`.
</ResponseField>

<Steps titleSize="h3">
  <Step title="Request the first page">
    ```bash theme={"system"}
    curl -X POST https://api.unkey.com/v2/apis.listKeys \
      -H "Authorization: Bearer <root key>" \
      -H "Content-Type: application/json" \
      -d '{"apiId": "api_1234abcd", "limit": 50}'
    ```
  </Step>

  <Step title="Read the cursor">
    ```json theme={"system"}
    {
      "meta": { "requestId": "req_5678efgh" },
      "data": [ { "keyId": "key_1111aaaa" }, { "keyId": "key_2222bbbb" } ],
      "pagination": { "cursor": "key_3333cccc", "hasMore": true }
    }
    ```
  </Step>

  <Step title="Send it back until hasMore is false">
    ```bash theme={"system"}
    curl -X POST https://api.unkey.com/v2/apis.listKeys \
      -H "Authorization: Bearer <root key>" \
      -H "Content-Type: application/json" \
      -d '{"apiId": "api_1234abcd", "limit": 50, "cursor": "key_3333cccc"}'
    ```
  </Step>
</Steps>

Keep the other request fields the same on every page. If you change a filter partway through, the cursor no longer works.


This documentation is built and hosted on [Mintlify](https://mintlify.com), a developer documentation platform.
