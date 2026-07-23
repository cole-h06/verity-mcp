# Verity Protocol

## 1. Introduction

The Verity Protocol defines the interoperability contract between clients and Verity deployments.

The protocol defines the method for how structured assertions are represented and transformed into privacy-preserving identifiers. Identifiers are transmitted and processed to produce deterministic credibility signals.

The protocol does not define the method for extracting structured assertions from unstructured data, nor does it prescribe a specific credibility inference algorithm. Those implementation details are outside the scope of this specification.

---

## 2. Requirement Language

The key words **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** in this document are to be interpreted as described in RFC 2119.

---

## 3. Core Concepts

A **Source** represents an independent origin that asserts one or more claims.

Examples could include websites, databases, organizations, documents, APIs, or other identifiable publishers.

---

### Assertion

An **Assertion** is the relationship between a source and a claim.

Assertions are the fundamental edges of the Verity credibility graph.

---

### Claim

A **Claim** represents a canonical statement describing a piece of structured information.

Multiple independent sources may assert the same claim.

---

### Client

A **Client** is software responsible for constructing protocol-compliant requests.

---

## 4. Design Principles

### Domain Agnostic

The protocol does not depend on any particular application domain.

Clients MAY submit assertions describing any type of structured information.

---

### Deterministic

Equivalent canonical assertions MUST produce equivalent linkage identifiers.

Given an identical graph state, a deployment MUST produce deterministic inference results.

---

### Content Blind

Verity deployments operate exclusively on opaque identifiers.

Deployments MUST NOT require semantic interpretation of the underlying content.

---

### Privacy Preserving

Clients are responsible for transforming canonical assertions into privacy-preserving linkage identifiers prior to transmission.

Deployments operate exclusively on those identifiers.

---

### Deployment Independent

The protocol supports both persistent and ephemeral deployments.

Persistence behavior is determined by the deployment implementation rather than the protocol itself.

---

## 5. Protocol Roles

### Client Responsibilities

A conforming client MUST:

- Extract structured assertions from local data.
- Canonicalize assertions according to the Verity Canonicalization Specification.
- Generate claim linkage identifiers.
- Generate attribute linkage identifiers.
- Construct protocol-compliant assertion messages.
- Submit requests to a Verity deployment.

A client MUST NOT require a deployment to perform semantic extraction, canonicalization, or fuzzy matching.

---

### Deployment Responsibilities

A conforming deployment MUST:

- Validate incoming protocol messages.
- Treat all linkage identifiers as opaque values.
- Resolve linkage identifiers within the local graph state.
- Perform credibility inference.
- Return protocol-compliant responses.

A deployment MUST NOT require raw assertion content in order to execute inference.

---

## 6. Processing Model

The protocol follows the following conceptual workflow.

```
Client

    Structured Extraction
            │
            ▼
     Canonicalization
            │
            ▼
   Linkage Generation
            │
            ▼
  Protocol Message Construction
            │
            ▼
     Request Submission

────────────────────────────────────────

Verity Deployment

      Message Validation
            │
            ▼
     Identifier Resolution
            │
            ▼
      Graph Integration
            │
            ▼
    Credibility Inference
            │
            ▼
      Response Generation
```

This workflow describes the logical stages of the protocol.

Implementations MAY optimize or reorder internal processing provided externally observable protocol behavior remains unchanged.

---

## 7. Protocol Requirements

A conforming implementation MUST satisfy the following requirements.

### Canonicalization

Clients MUST canonicalize assertions before generating linkage identifiers.

Equivalent canonical assertions MUST generate equivalent identifiers.

---

### Linkage Identifiers

Clients MUST generate linkage identifiers according to the Verity Canonicalization Specification.

Deployments MUST treat linkage identifiers as opaque identifiers.

Deployments MUST NOT infer or depend upon their underlying contents.

---

### Assertions

Every protocol assertion MUST contain:

- a source identifier
- an attribute linkage identifier
- a claim linkage identifier

The exact message format is defined by `api.md`.

---

### Inference

Deployments MUST produce deterministic credibility results for an identical graph state.

The protocol does not prescribe any particular inference algorithm.

---

### Responses

Deployments MUST return responses conforming to the protocol response schema defined in `api.md`.

---

## 8. Compatibility

Future protocol versions MAY introduce additional optional fields without breaking existing implementations.

Inference algorithms MAY evolve independently of the protocol provided they continue to satisfy the guarantees defined by this specification.

Changes that modify protocol message formats or required behavior constitute a new protocol version.

---

## 9. Security Considerations

The Verity Protocol is designed to minimize disclosure of underlying assertion content by operating on privacy-preserving linkage identifiers.

Implementations remain responsible for transport security, authentication, authorization, access control, and cryptographic key management.

These concerns are outside the scope of this specification.