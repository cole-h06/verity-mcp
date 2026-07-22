# API Reference

The Verity API computes structural credibility over networks consisting of sources, claims, and assertions.

The specific request and response schemas exposed by Verity deployments are defined in this document. API behavior, including versioning, identifiers, errors, and tool interfaces, are also defined.

The API is transport-independent. The reference implementation is exposed over the [Model Context Protocol (MCP)](https://modelcontextprotocol.io), but the schemas defined in this document may be used by any compliant deployment.

To see related specifications, visit:

- **[`protocol.md`](protocol.md)** —The communication contract between clients and Verity deployments. Defines the JSON-RPC protocol and request/response formats.
- **[`architecture.md`](architecture.md)**  — Describes the high-level system architecture and component interactions.
- **[`canonicalization.md`](canonicalization.md)** - Deterministic normalization rules. Specifies how semantically equivalent assertions produce the same graph representation before inference.
- **[`sdk.md`](sdk.md)** - Client integration guides for constructing graphs, linkage token generation, submitting requests from supported languages.
- **[`mcp.md`](mcp.md)** — The MCP binding for the Verity Protocol.

---

# Versioning

The API distinguishes between protocol versions and algorithm versions.

| Property | Description |
| --- | --- |
| `protocol_version` | The API contract a deployment implements. |
| `algorithm_version` | The credibility inference implementation used to produce results. |

Protocol versions define the behavior and structure of the API.

Algorithm versions define the inference implementation. They may be subject to change independently of the protocol version, provided the API remains backward compatible.

```json
{
  "protocol_version": "v1",
  "algorithm_version": "verity-v1"
}
```

---

# Authentication

Authentication requirements are deployment-specific.

Self-hosted deployments MAY expose unauthenticated interfaces.

Hosted deployments MAY require authentication before processing requests.

---

# Errors

Protocol-level failures are returned as JSON-RPC 2.0 error objects.

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "error": {
    "code": -32602,
    "message": "The submitted assertion graph is invalid.",
    "data": {
      "verity_code": "INVALID_GRAPH",
      "details": {}
    }
  }
}
```

| Property | Type | Description |
| --- | --- | --- |
| `error.code` | integer | Standard JSON-RPC error code. |
| `error.message` | string | Human-readable error description. |
| `error.data` | object | Verity-specific error information. |
| `error.data.verity_code` | string | Machine-readable Verity error code. |
| `error.data.details` | object | Additional error details. |

## Error Codes

| Code | Description |
| --- | --- |
| `INVALID_REQUEST` | The request does not conform to the required schema. |
| `INVALID_GRAPH` | The submitted assertion graph is malformed or cannot be processed. |
| `UNKNOWN_CLAIM` | The requested claim is not available to the deployment. |
| `UNSUPPORTED_PROTOCOL_VERSION` | The requested protocol version is not supported. |
| `INFERENCE_FAILURE` | The inference run could not be completed. |

---

# Tools

The API defines the following tools.

| Tool | Description |
| --- | --- |
| `ping` | Returns information about the deployment. |
| `infer_credibility` | Computes structural credibility over an assertion graph. |
| `trace_claim` | Returns structural information associated with a claim. |

---

# ping

Returns information about the deployment.

## Request

```json
{}
```

## Request Body

The `ping` tool does not accept any request properties.

## Response

```json
{
  "status": "ok",
  "protocol_version": "v1",
  "algorithm_version": "verity-v1"
}
```

## Response Body

| Property | Type | Description |
| --- | --- | --- |
| `status` | string | Deployment status. |
| `protocol_version` | string | Protocol version a deployment implements. |
| `algorithm_version` | string | Inference implementation a deployment implements. |

---

# infer_credibility

Computes structural credibility over an assertion graph.

## Request

```json
{
  "assertions": [
    {
      "source_id": "src_7d9a4e...",
      "claim_id": "clm_f72d10..."
    },
    {
      "source_id": "src_b28491...",
      "claim_id": "clm_f72d10..."
    },
    {
      "source_id": "src_91c8fd...",
      "claim_id": "clm_51d8ab..."
    },
    {
      "source_id": "src_e46ab2...",
      "claim_id": "clm_f72d10..."
    }
  ]
}
```

## Request Body

| Property | Type | Description |
| --- | --- | --- |
| `assertions` | array | Collection of source-to-claim assertions. |

### assertions

| Property | Type | Description |
| --- | --- | --- |
| `source_id` | string | Opaque source identifier. |
| `claim_id` | string | Opaque claim identifier. |

## Response

```json
{
  "summary": {
    "source_count": 4,
    "claim_count": 2,
    "assertion_count": 4
  },
  "source_credibility": [
    {
      "source_id": "src_7d9a4e...",
      "credibility": 0.29
    },
    {
      "source_id": "src_b28491...",
      "credibility": 0.27
    },
    {
      "source_id": "src_91c8fd...",
      "credibility": 0.17
    },
    {
      "source_id": "src_e46ab2...",
      "credibility": 0.27
    }
  ],
  "claim_support": [
    {
      "claim_id": "clm_f72d10...",
      "support": 0.83,
      "independent_support_count": 3,
      "dependent_support_count": 0,
      "is_attribute_max_support": true,
      "conflicting_claims": [
        {
          "claim_id": "clm_51d8ab...",
          "support": 0.17
        }
      ]
    },
    {
      "claim_id": "clm_51d8ab...",
      "support": 0.17,
      "independent_support_count": 1,
      "dependent_support_count": 0,
      "is_attribute_max_support": false,
      "conflicting_claims": [
        {
          "claim_id": "clm_f72d10...",
          "support": 0.83
        }
      ]
    }
  ],
  "source_dependencies": [
    {
      "source_a": "src_7d9a4e...",
      "source_b": "src_b28491...",
      "shared_claim_count": 1
    },
    {
      "source_a": "src_7d9a4e...",
      "source_b": "src_e46ab2...",
      "shared_claim_count": 1
    },
    {
      "source_a": "src_b28491...",
      "source_b": "src_e46ab2...",
      "shared_claim_count": 1
    }
  ],
  "metadata": {
    "algorithm_version": "verity-v1",
    "iterations": 37,
    "converged": true,
    "computation_time_ms": 8
  }
}
```

## Response Body

| Property | Type | Description |
| --- | --- | --- |
| `summary` | object | Statistics describing the submitted graph. |
| `source_credibility` | array | Credibility scores for each source. |
| `claim_support` | array | Support scores for each claim. |
| `source_dependencies` | array | Shared-claim relationships between sources. |
| `metadata` | object | Information about the completed inference run. |

### summary

| Property | Type | Description |
| --- | --- | --- |
| `source_count` | integer | Number of unique sources. |
| `claim_count` | integer | Number of unique claims. |
| `assertion_count` | integer | Number of processed assertions. |

### source_credibility

| Property | Type | Description |
| --- | --- | --- |
| `source_id` | string | Opaque source identifier. |
| `credibility` | number | Computed credibility score. |

### claim_support

| Property | Type | Description |
| --- | --- | --- |
| `claim_id` | string | Opaque claim identifier. |
| `support` | number | Computed support score. |
| `independent_support_count` | integer | Number of independent supporting sources. |
| `dependent_support_count` | integer | Number of dependent supporting sources. |
| `is_attribute_max_support` | boolean | Identifies if a claim has the highest support among competing claims for the same attribute. |
| `conflicting_claims` | array | Competing claims and their corresponding support scores. |

### source_dependencies

| Property | Type | Description |
| --- | --- | --- |
| `source_a` | string | First source identifier. |
| `source_b` | string | Second source identifier. |
| `shared_claim_count` | integer | Number of claims asserted by both sources. |

### metadata

| Property | Type | Description |
| --- | --- | --- |
| `algorithm_version` | string | Inference implementation version. |
| `iterations` | integer | Number of inference iterations. |
| `converged` | boolean | Identifies if convergence was reached. |
| `computation_time_ms` | integer | Total execution time (milliseconds). |

---

# trace_claim

Returns structural information associated with a claim.

## Request

```json
{
  "claim_id": "clm_f72d10..."
}
```

## Request Body

| Property | Type | Description |
| --- | --- | --- |
| `claim_id` | string | Opaque claim identifier. |

## Response

```json
{
  "claim_id": "clm_f72d10...",
  "support": 0.83,
  "independent_support_count": 3,
  "dependent_support_count": 0,
  "is_attribute_max_support": true,
  "supporting_sources": [
    {
      "source_id": "src_7d9a4e...",
      "credibility": 0.29
    },
    {
      "source_id": "src_b28491...",
      "credibility": 0.27
    },
    {
      "source_id": "src_e46ab2...",
      "credibility": 0.27
    }
  ],
  "conflicting_claims": [
    {
      "claim_id": "clm_51d8ab...",
      "support": 0.17
    }
  ]
}
```

## Response Body

| Property | Type | Description |
| --- | --- | --- |
| `claim_id` | string | Opaque claim identifier. |
| `support` | number | Computed support score. |
| `independent_support_count` | integer | Number of independent supporting sources. |
| `dependent_support_count` | integer | Number of dependent supporting sources. |
| `is_attribute_max_support` | boolean | Identifies if the claim has the highest support among competing claims for the same attribute. |
| `supporting_sources` | array | Sources asserting the claim and their credibility scores. |
| `conflicting_claims` | array | Competing claims and their corresponding support scores. |

### supporting_sources

| Property | Type | Description |
| --- | --- | --- |
| `source_id` | string | Opaque source identifier. |
| `credibility` | number | Computed credibility score. |

### conflicting_claims

| Property | Type | Description |
| --- | --- | --- |
| `claim_id` | string | Opaque claim identifier. |
| `support` | number | Computed support score. |

---

# MCP Transport Example

Example of an `infer_credibility` request transported over the Model Context Protocol (MCP):

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "infer_credibility",
    "arguments": {
      "assertions": [
      {
        "source_id": "src_7d9a4e...",
        "claim_id": "clm_f72d10..."
      },
      {
        "source_id": "src_b28491...",
        "claim_id": "clm_f72d10..."
      },
      {
        "source_id": "src_91c8fd...",
        "claim_id": "clm_51d8ab..."
      },
      {
        "source_id": "src_e46ab2...",
        "claim_id": "clm_f72d10..."
      }
    ]
  }
}
```

A successful response returns the tool result in the `result` field.

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "...": "..."
  }
}
```

Transport-specific behavior, including request identifiers, notifications, and error handling, is defined in `mcp.md`.
