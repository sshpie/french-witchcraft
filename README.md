
<p align="center">
  <b>French Witchcraft</b>
</p>

<p align="center">
  30-phase autonomous API reverse engineering tool.
</p>

---

Point it at any HTTP API. It maps the surface, extracts schema, and probes for vulnerabilities — no prior knowledge required.

It autonomously:Reverse-engineers unknown HTTP APIs
Maps endpoints and schemas
Actively probes for a wide range of vulnerabilities (injection, JWT attacks, BOLA/IDOR, BFLA, mass assignment, SSRF, NoSQL injection, OAuth issues, etc.)



## Install

```bash
pip install -e .
```

## Usage

```
frwitch http://target:port
frwitch http://target:port --depth deep
frwitch http://target:port --depth deep --output report.json
frwitch http://target:port --focus jwt
frwitch http://target:port --focus nosql
frwitch http://target:port --focus bfla
frwitch http://target:port --focus oauth2
frwitch http://target:port --fuzz-params /api/v1/users
```

## Phases

| # | Phase | What it does |
|---|-------|-------------|
| 1 | Framework detection | Fingerprints FastAPI / Flask / Django / Gradio / Express / Spring via headers and known paths |
| 2 | OpenAPI harvest | Pulls schema from 16 known spec paths; extracts all endpoints, parameters, auth schemes |
| 2b | Version discovery | Probes /v1–/v5 and /api/v1–/api/v5 variants |
| 3 | Path fuzzing | 80-path wordlist × extension variants; maps live surface |
| 4 | Injection | Path traversal, pickle RCE, SQLi, SSTI, SSRF, XXE, format string, prototype pollution |
| 4b | Verb tampering | 12 HTTP methods + X-HTTP-Method-Override headers |
| 4c | Content-type switching | JSON → form → XML → multipart per endpoint |
| 4d | Parameter pollution | HPP via duplicate keys and array notation |
| 4e | Mass assignment | 20 privileged field names (role, admin, is_superuser, etc.) |
| 4f | BOLA / IDOR | Integer, UUID, alphanumeric ID enumeration across resource paths |
| 4g | JWT analysis | alg:none forge, weak secret crack, kid/jku/x5u injection |
| 4h | Hidden params | JS bundle extraction via regex patterns |
| 5 | Error extraction | Stack traces, path disclosure, 422 validation error leakage |
| 6 | State mapping | Identifies non-idempotent endpoints |
| 7 | Rate limiting | 20-request burst; 429 detection |
| 8 | OAuth2 / OIDC | Discovery, redirect_uri manipulation, default client creds, JWKS probing |
| 9 | gRPC | gRPC-Web detection, server reflection paths |
| 10 | GraphQL advanced | Batching, alias flooding, depth bombing, field suggestions, introspection |
| 11 | Rate limit bypass | X-Forwarded-For / CF-Connecting-IP spoofing, path variation |
| 12 | SOAP / WSDL | WSDL discovery, XXE injection in SOAP body |
| 13 | JSON-RPC | system.listMethods, batch execution |
| 14 | Business logic | Numeric boundaries, status/role privilege escalation |
| 15 | Webhook SSRF | Cloud metadata URLs via webhook parameter injection |
| 16 | Default creds | Auth endpoint spray with common credential pairs |
| 17 | CORS | Origin reflection, null origin bypass, wildcard + credentials |
| 18 | JWT alg confusion | RSA public key exposure; RS256 → HS256 algorithm confusion surface |
| 19 | NoSQL injection | MongoDB $ne / $gt / $regex / $where operator injection; auth bypass |
| 20 | BFLA | Admin path enumeration; privileged operation suffix probing |
| 21 | Data exposure | PII scanner: email, SSN, CC, phone, API keys, AWS keys, private keys |
| 22 | Host header injection | Origin reflection, password reset link poisoning, override headers |
| 23 | Timing oracle | User enumeration via bcrypt-path response time delta |
| 24 | Second-order injection | Store payload via write endpoint; trigger via read endpoint |
| 25 | Null byte / WAF evasion | %00 path and parameter truncation, double-encode bypass |
| 26 | OAuth dynamic reg | RFC7591 unauth client registration; token introspection (RFC7662); revocation (RFC7009) |
| 27 | WebSocket / SSE | WS upgrade detection, GraphQL subscription probe, Server-Sent Events |
| 28 | Pagination traversal | Follows Link rel=next headers; bulk record enumeration (BOLA at scale) |
| 29 | Shadow versions | Sunset/Deprecation headers; /v0 /beta /legacy /internal path probing |
| 30 | OAuth state / PKCE | Missing state parameter (CSRF); plain code_challenge_method downgrade; open redirect_uri |

## Focus flags

`--focus` limits the run to one dimension. Useful when you already know the target surface.

```
injection   nosql    evasion    graphql    grpc
schema      bfla     pii        websocket  soap
state       cors     timing     shadow     biz
jwt         oauth2   ssrf       bola
auth        creds
```

## Depth

| Flag | Phases active |
|------|--------------|
| `quick` | 1, 2, 3 only — fast surface map |
| `normal` (default) | All phases except timing oracle, pagination traversal, second-order injection |
| `deep` | All 30 phases |

## Output

Default: structured stdout. With `--output report.json`: JSON report with endpoint map, inferred parameters, and vulnerability signals.

## Requirements

```
Python 3.8+
requests
```

---

For authorized security testing only.
