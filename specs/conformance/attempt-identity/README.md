# Known-answer vectors for section 5.5

Section 5.5 of the v2 specification (Settlement and Refusal Identity Invariant) defines how the request members that enter an attempt identity are canonicalized. These vectors pin the result for seven raw requests, so an implementation in any language can check itself without a shared checker.

Each input comes as a pair of files:

- `known-answer-<name>.json` records the value the section prescribes. Expected outcome: `PASS`.
- `known-answer-<name>-wrong.json` records a plausible wrong value for the same input. Expected outcome: `FAIL` with `CANONICAL_MEMBER_MISMATCH`.

An implementation passes a file when it canonicalizes `input.known_answers[].raw` and compares the result to `observed` the way `expected` says: equal gives `PASS`, different gives `FAIL`.

| Input | Pins |
| --- | --- |
| `query-uppercase-hex-and-sort` | percent-encoding with uppercase hex, pairs sorted |
| `query-sort-by-value` | repeated names kept, sorted by encoded value |
| `query-plus-stays-plus` | a literal `+` is not read as a space |
| `body-json-rfc8785` | JSON body canonicalized with RFC 8785, then SHA-256 |
| `body-gzip-then-rfc8785` | `Content-Encoding` reversed before canonicalization |
| `body-iso-8859-1-is-latin1` | each byte of an ISO-8859-1 body is the code point of the same number |
| `body-duplicate-keys-hashed-as-bytes` | duplicate object keys (also after unescaping): the decoded bytes are hashed, no canonicalization |

The query string and body fields in `raw` are the wire form: `query` is the raw query string, `bodyBase64` the body as received, `contentEncoding` and `contentType` the headers.

## Provenance

The right values were computed by two implementations written separately, one in JavaScript and one in Python using only its standard library. They share no code and agree on all seven inputs. A third implementation, written separately in Python with only the standard library and run by ruiruii, matches all seven as well (https://github.com/x402-foundation/x402/pull/3776#issuecomment-6419277292). The files are copied byte for byte from `vectors/attempt-identity/` in https://github.com/holistis/agent-receipt-verify at commit `9a356bd`, which also holds the vectors that record what an implementation did rather than pin a value. All files carry their own `provenance` and `limits` fields.

## Limits

- These vectors pin canonical members, not a whole identity. They say nothing about how the members are combined or hashed into one.
- Two points the text of 5.5 leaves open are not pinned here: empty segments in a query (`a=1&&b=2`) and a `%` that is not followed by two hex digits.
