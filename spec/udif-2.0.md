# UDIF 2.0 Specification

**Status:** Draft  
**Version:** 2.0.0  
**Invented by:** Angela Benton  
**Original standard:** Streamlytics, Inc. (2018)  
**License:** Apache 2.0  
**Schema:** `/spec/schema/udif.schema.json`

---

## Overview

UDIF (Universal Data Interchange Format) is an open standard for 
portable, sovereign data. Version 2.0 extends the original standard 
into the AI interaction layer.

A UDIF file is a structured JSON document that represents a person's 
data in a format that is:

- **Platform-agnostic** — readable by any system without permission
- **Self-contained** — no platform dependency to access or interpret
- **Consent-forward** — consent type is captured in the format itself
- **Verifiable** — provenance chain proves origin and integrity
- **Human-centered** — captures emotional, energetic, and values context alongside behavioral data
- **Extensible** — modular structure supports any data type

---

## Required Fields

Every valid UDIF 2.0 file must contain:

- `udif` — version identifier
- `meta` — source, session, timestamp, and consent
- `platform` — origin platform details
- `data_event` — the interaction or data being captured

All other modules are optional but recommended for richer portability.

---

## Modules

### `udif`
Version string. Always `"2.0"`. Identifies the file as UDIF and 
specifies the specification version.

---

### `meta`
Session-level metadata about the data capture.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| source | string | yes | Platform or system generating the file |
| generator | string | no | Specific model or tool used |
| session_id | string | yes | Unique session identifier |
| timestamp | datetime | yes | ISO 8601 timestamp of capture |
| user_id | string | no | User identifier on the source platform |
| consent_granted | boolean | yes | Whether user has consented to this capture |
| consent_type | string | no | `explicit`, `contextual`, or `implied` |
| data_type | string | no | Category of data being captured |

---

### `identity`
Information about the data subject. All fields are optional to 
protect privacy while enabling personalization.

| Field | Type | Description |
|-------|------|-------------|
| alias | string | Pseudonym or display name |
| public_key | string | Cryptographic public key for verification |
| public_key_format | string | Format of the public key: `jwk`, `pem`, or `base64` |
| persona_tags | array | Self-described identity tags |
| archetype | string | Self-described archetype or role |
| language | string | Preferred language code (BCP 47, e.g. `en-US`) |
| timezone | string | IANA timezone string |

#### Public Key Format

When `public_key` is provided, `public_key_format` SHOULD also be 
provided to enable interoperability. Supported formats:

- **`jwk`** — JSON Web Key (RFC 7517). The `public_key` field contains 
  a JSON-serialized JWK object. This is the RECOMMENDED format for new 
  implementations because it is self-describing (includes algorithm and 
  key type metadata).
- **`pem`** — PEM-encoded public key (RFC 7468). The `public_key` field 
  contains the PEM string including `-----BEGIN PUBLIC KEY-----` headers.
- **`base64`** — Raw public key bytes, base64-encoded. When using this 
  format, the key type and algorithm must be established out of band.

If `public_key_format` is omitted, consumers SHOULD attempt JWK parsing 
first, then PEM detection, before falling back to raw base64.

---

### `context`
Environmental and state context at the time of the data event. 
Captures the human conditions surrounding the data, not just 
the data itself.

| Field | Type | Description |
|-------|------|-------------|
| location | object | Latitude, longitude, and optional geo label |
| device | object | Device type and operating system |
| emotional_state | array | Self-reported or inferred emotional states |
| presence_level | string | `low`, `medium`, or `high` |
| energy_score | integer | 1–10 self-reported energy level |

---

### `platform`
Details about the source platform and export format.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| name | string | yes | Platform name (see naming conventions below) |
| data_format | string | yes | Format of the original export |
| source_type | string | yes | Type of data source |
| export_date | datetime | yes | When the data was exported |
| raw_schema_reference | string | no | URL to the platform's original schema |
| session_reference_id | string | no | Platform's own session identifier |

#### Platform Naming Conventions

The `platform.name` field is an open string to avoid requiring schema 
updates when new platforms emerge. To maintain consistency across 
implementations, converters SHOULD use the platform's official product 
name as it appears in the platform's own documentation:

| Platform | Recommended Value |
|----------|------------------|
| OpenAI (ChatGPT) | `"OpenAI"` |
| Google (Gemini) | `"Google"` |
| Anthropic (Claude) | `"Anthropic"` |
| Perplexity | `"Perplexity"` |
| Mistral | `"Mistral"` |
| xAI (Grok) | `"xAI"` |
| Meta (Llama-based) | `"Meta"` |
| Cohere | `"Cohere"` |
| Local/self-hosted | `"Local"` or the specific runtime name (e.g. `"Ollama"`, `"LM Studio"`, `"vLLM"`) |
| Custom applications | A descriptive name for the application |

This table is informational, not normative. Any string value is valid.

Supported data formats: `json`, `html`, `markdown`, `zip`, `csv`, 
`proprietary`

Supported source types: `chat_log`, `file`, `search`, `audio`, 
`image`, `video`, `code`, `multimodal`

---

### `data_event`
The core interaction or data being captured. For AI interactions 
this includes the full message thread.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| type | string | yes | Type of event |
| service | string | yes | Service within the platform |
| tags | array | no | Descriptive tags |
| content_title | string | no | Human-readable title for the event |
| duration | integer | no | Duration in seconds |
| value_perceived | integer | no | User-perceived value, 1–10 |
| is_shareable | boolean | no | Whether user consents to sharing |
| messages | array | no | Individual messages in the interaction |
| shared_links | array | no | Links shared during the interaction |

#### Message Object

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| role | string | yes | Message role (see below) |
| content | string or array | yes | Message content (see below) |
| timestamp | datetime | yes | ISO 8601 timestamp |
| name | string | no | Identifier for multi-agent scenarios |
| model | string | no | Model that generated this message (e.g. `gpt-4o`, `claude-sonnet-4-5-20250929`) |
| tool_call_id | string | no | Correlation ID linking a tool result to its tool call |
| attachments | array | no | File references associated with this message |

##### Roles

| Role | Description |
|------|-------------|
| `user` | Message from the human user |
| `assistant` | Response from the AI model |
| `system` | System-level instructions or context (typically not shown to users) |
| `tool` | Result returned from a tool/function call |
| `developer` | Developer-provided instructions in API-based interactions |

Implementations encountering roles not in this list SHOULD preserve 
them as-is rather than mapping them to a known role or discarding them. 
This ensures lossless round-tripping through platforms that define 
custom roles.

##### Content

The `content` field supports two forms:

**Simple string** — for plain text messages:
```json
{
  "role": "user",
  "content": "What does real data portability look like?",
  "timestamp": "2026-03-01T14:00:00Z"
}
```

**Array of content blocks** — for structured or multimodal messages:
```json
{
  "role": "assistant",
  "content": [
    {
      "type": "text",
      "text": "Here is the analysis you requested."
    },
    {
      "type": "image",
      "source": "generated",
      "media_type": "image/png",
      "description": "Chart showing data portability adoption over time"
    }
  ],
  "timestamp": "2026-03-01T14:00:03Z"
}
```

Defined content block types:

| Type | Required Fields | Optional Fields | Description |
|------|----------------|-----------------|-------------|
| `text` | `text` | — | Plain text content |
| `image` | — | `source`, `media_type`, `url`, `description`, `data` | Image content or reference |
| `file` | `filename` | `media_type`, `url`, `data` | File attachment |
| `tool_call` | `tool_name`, `tool_input` | `tool_call_id` | Invocation of a tool/function |
| `tool_result` | `tool_call_id` | `content`, `is_error` | Result returned from a tool |
| `code_execution` | `code` | `language`, `output`, `exit_code` | Code that was executed |

Converters SHOULD use the simple string form when the message is 
plain text only. The array form SHOULD be used when the message 
contains multiple content types, non-text content, or structured 
tool interactions.

##### Attachments

The `attachments` array contains references to files associated 
with a message:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| filename | string | yes | Original filename |
| media_type | string | no | MIME type |
| size_bytes | integer | no | File size |
| url | string | no | URL or path to the file |

---

### `values`
Explicit user-defined permissions and preferences governing how 
their data can be used.

| Field | Type | Description |
|-------|------|-------------|
| data_use_permissions | array | Permitted uses of this data |
| sharing_boundaries | array | Explicit limits on sharing |
| preferred_exchanges | array | Value exchanges the user accepts |
| core_values | array | User's stated core values |

---

### `frequency`
Captures the human signal beneath the data — the energetic and 
creative context that purely behavioral formats miss. This module 
is unique to UDIF.

| Field | Type | Description |
|-------|------|-------------|
| intended_impact | string | What the user intended to achieve |
| creative_source | string | Origin of the creative or intellectual energy |
| self_expression_level | string | `low`, `medium`, or `high` |
| authenticity_score | integer | 1–10 self-reported authenticity |
| energetic_signature | string | Unique hash representing this session's energy pattern |

---

### `provenance`
Cryptographic metadata establishing the origin, authenticity, 
and integrity of the file. The `chain` array is what makes UDIF 
verifiable — not just portable.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| created_at | datetime | yes | When this UDIF file was created |
| updated_at | datetime | no | When this file was last modified |
| source | string | yes | Originating platform |
| hash | string | no | SHA-256 hash of the data_event object |
| signature | string | no | Cryptographic signature proving authorship |
| chain | array | no | Auditable history of platform transfers |

**Chain object:**

| Field | Type | Description |
|-------|------|-------------|
| platform | string | Platform name at this point in the chain |
| exported_at | datetime | When data left this platform |
| hash | string | Hash of data at export, enabling tamper detection |

The chain creates an auditable trail from origin through every 
platform the data has touched. Each hash allows any system to 
verify the data hasn't been altered in transit.

---

### `raw_payload`
The original unmodified data from the source platform, preserved 
alongside the standardized UDIF structure. Ensures nothing is lost 
in translation.

### `raw_payload_ref`
A URI reference to the raw payload if stored externally rather 
than inline.

---

## Versioning

UDIF follows semantic versioning. The `udif` field in every file 
identifies the version of the spec it conforms to. Breaking changes 
increment the major version.

---

## The Frequency Module

The `frequency` module deserves additional context because it has 
no equivalent in any other data standard.

Most data formats capture what happened. UDIF captures what it 
meant to the person it happened to. The frequency module was 
designed on the premise that human data has an energetic quality 
that behavioral data alone cannot represent — the intention behind 
an interaction, the creative state it came from, the authenticity 
of the expression.

This is not metadata in the traditional sense. It is human signal.

---

## Contributing

See `CONTRIBUTING.md` for guidelines on contributing to the 
specification.

---

## References

- Patent: US20220300636A1 — *System and Method for Standardizing Data*
- Inventor: [Angela Benton](https://angelabenton.com)
- Protocol: [Heirloom](https://yourheirloom.ai)
