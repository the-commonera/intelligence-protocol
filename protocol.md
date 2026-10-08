# Intelligence Protocol Specification

**Version:** 1.0.0 (draft)
**Envelope:** JSON-RPC 2.0
**Scope:** `workflow.run`, `template.register`, `rpc.discover`

---

## 1. Transport

| Item | Value |
|---|---|
| Endpoint | `POST /rpc` |
| Content-Type | `application/json` |
| Encoding | UTF-8 |
| Batching | Supported per JSON-RPC 2.0 (array of requests returns array of responses) |
| Notifications | Requests without `id` receive no response |

## 2. Authentication and headers

Two kinds of keys are sent as HTTP headers. **No secrets are sent in `params`.**

| Header | Required | Purpose |



|---|---|---|


| `X-LLM-key-<modelname>: <llm-api-key>` | yes | llm api key |
| `X-MCP-Key-<capability-name>: <mcp-api-key>` | API key the server uses to call that capability's MCP endpoint |




## 3. Message format

### Request

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "workflow.run",
  "params": { }
}
```

- `jsonrpc` MUST be `"2.0"`.
- `id` is a string or number
- `params` MUST be an object (named parameters)



### Success response

```json
{ "jsonrpc": "2.0", "id": 1, "result": { } }
```

### Error response

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "error": { "code": -32602, "message": "Invalid params", "data": [] }
}
```

---

## 4. Methods

### 4.1 `workflow.run`

Runs a workflow by name.

**Params**

| Name | Type | Required | Description |

|---|---|---|---|
| `name` | string | yes | Name of the workflow/template to run |
| `goal` | string | no | Objective for this run |
| `provider` | string | no | LLM provider identifier |
| `model` | string | no | Model identifier |
| `context` | object | no | Free-form context passed to the workflow |

Additional properties are allowed and passed through to the workflow.


**Result** *(proposed)*

| Name | Type | Description |
|---|---|---|
| `runId` | string | Unique run identifier |
| `status` | string | `queued` \| `running` \| `succeeded` \| `failed` |
| `output` | any | Workflow output, present when `status` is `succeeded` |


**Example**

```json
// request
{"jsonrpc":"2.0","id":1,"method":"workflow.run",
 "params":{"name":"","goal":"","provider":"","model":"","context":{}}}

// response
{"jsonrpc":"2.0","id":1,"result":{"runId":"run_123","status":"succeeded","output":"..."}}

```

---

### 4.2 `template.register`

Registers a workflow template. Maps to `POST /templates/register`.

**Params**

| Name | Type | Required | Constraints |
|---|---|---|---|
| `name` | string | yes | |
| `version` | string | yes | Semver `MAJOR.MINOR.PATCH`, pattern `^\d+\.\d+\.\d+$` |
| `description` | string | yes | `minLength: 1` |
| `capabilities` | array\<Capability\> | yes | `minItems: 1` |
| `context` | object | yes | |
| `modeldata` | object | yes | |
| `webhook_url` | string | yes | Valid URI |

Additional properties are allowed.




**Result** *(proposed)*

| Name | Type | Description |
|---|---|---|
| `ok` | boolean | `true` on success |
| `name` | string | Registered template name |
| `version` | string | Registered version |

**Example**

```json
coming soon
```


---

### 4.3 `rpc.discover`

Returns the method catalog with parameter schemas

**Params:** none (`{}` or omitted)

**Result:** object keyed by method name.

```json
{
  "workflow.run":      { "params": { "type": "object", "required": ["name"], "properties": {} } },
  "template.register": { "params": { "type": "object", "required": ["name","version","description","capabilities","context","modeldata","webhook_url"], "properties": {} } }
}
```

---




## 5. Versioning

The protocol uses semver. Breaking changes bump the major version. Clients MAY discover the supported version via `rpc.discover` (a `version` field will be added in a later minor release).

## 8. Open items

- [ ] Finalize result schemas for `workflow.run` and `template.register`
- [ ] Decide sync vs. async execution for long-running workflows (`runId` plus `workflow.status`)
- [ ] Decide how `webhook_url` callbacks map into the protocol (push events vs. webhook)
