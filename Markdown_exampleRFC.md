# RFC 0042: Unified Authentication Service 🔐

**Status**: Draft
**Author**: Engineering Team
**Created**: 2026-04-23
**Target Release**: Q3 2026

## Summary

- **🎯 Goal**: Consolidate three legacy auth systems into a single **unified authentication service**
- **📦 Scope**: Web, mobile, and internal tooling
- **🔄 Migration**: Phased rollout over two quarters with zero downtime
- **🛡️ Security**: OAuth 2.1 + OIDC with hardware key support
- **⚡ Performance**: Target p99 latency under 50ms for token validation
- **📊 Observability**: Full tracing, structured logs, and SLO dashboards from day one

### Motivation

Our current authentication landscape is fragmented across three services built at different times by different teams. Each has its own session model, token format, and user database. This creates real cost: duplicated bugs, inconsistent security posture, and a login experience that confuses users when they cross product boundaries.

| System | Origin | Users | Pain Points |
|--------|--------|-------|-------------|
| **LegacyAuth v1** | 2018 | Web app | Session cookies only, no MFA |
| **MobileAuth** | 2020 | iOS/Android | Custom JWT format, manual refresh |
| **InternalSSO** | 2022 | Staff tools | SAML, hard to extend |

### Non-Goals

This RFC does not cover:

- Migrating the underlying user identity store (tracked in RFC 0039)
- Changes to authorization or RBAC (see RFC 0041)
- Social login providers beyond Google and Apple
- Passwordless email magic links (future work)

### Proposed Architecture

The new service sits behind an API gateway and exposes a small, well-defined surface. Clients obtain tokens through standard OAuth flows, and downstream services verify tokens against a shared JWKS endpoint.

```python
# Token validation pseudocode
def validate_token(token: str) -> Claims:
    """Verify a JWT against the shared JWKS."""
    header = decode_header(token)
    key = jwks_cache.get(header["kid"])
    if not key:
        raise InvalidTokenError("unknown key id")
    claims = verify_signature(token, key)
    if claims.exp < now():
        raise ExpiredTokenError()
    return claims
```

```javascript
// Client refresh flow
async function refreshAccessToken(refreshToken) {
  const response = await fetch('/auth/v2/token', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      grant_type: 'refresh_token',
      refresh_token: refreshToken,
    }),
  });
  if (!response.ok) throw new Error('refresh failed');
  return response.json();
}
```

### API Surface

The public endpoints follow OAuth 2.1 conventions. All responses are JSON, all errors follow RFC 7807 problem details.

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/auth/v2/authorize` | GET | Start authorization code flow |
| `/auth/v2/token` | POST | Exchange code or refresh token |
| `/auth/v2/introspect` | POST | Validate and inspect a token |
| `/auth/v2/revoke` | POST | Revoke an active token |
| `/auth/v2/.well-known/jwks.json` | GET | Published signing keys |

### Security Considerations

Authentication changes carry real risk, so the design leans on established patterns rather than novel cryptography. Tokens are short-lived (15 minutes for access, 30 days for refresh), rotation is mandatory on every refresh, and all signing keys live in the HSM. Rate limits apply per client and per user. Audit logs capture every token mint, refresh, and revocation with a correlation ID that survives across services.

### Rollout Plan

Migration happens in four phases, each gated on success criteria from the previous one:

- **Phase 1 — Shadow mode**: Deploy the service, mirror all auth traffic, compare results
- **Phase 2 — Opt-in**: New signups and internal tooling move first, legacy stays live
- **Phase 3 — Forced migration**: Existing users migrate on next login, old tokens honored
- **Phase 4 — Decommission**: Turn off legacy endpoints, archive their databases

### Open Questions

A few things still need input from stakeholders:

- Do we need FAPI 2.0 conformance now, or can it wait for a later iteration?
- What is the right key rotation cadence for the JWKS?
- Should refresh tokens be bound to the device via DPoP, and if so, on which platforms?
- How do we handle the small population of users still on the SAML flow?

### Alternatives Considered

We looked at three alternatives before landing on this design:

| Option | Pros | Cons |
|--------|------|------|
| **Buy (Auth0, Okta)** | Fast, battle-tested | Cost at scale, data residency |
| **Extend LegacyAuth v1** | Lowest migration cost | Inherits architectural debt |
| **Build new (this RFC)** | Clean slate, fits our needs | Highest upfront engineering cost |

### Timeline

- **Week 1–2**: RFC review and approval
- **Week 3–6**: Service skeleton, JWKS, core token flows
- **Week 7–10**: Client SDKs for web and mobile
- **Week 11–14**: Shadow mode in staging, then production
- **Week 15+**: Phased migration per the rollout plan

### References

- [OAuth 2.1 Draft](https://datatracker.ietf.org/doc/draft-ietf-oauth-v2-1/)
- [OpenID Connect Core](https://openid.net/specs/openid-connect-core-1_0.html)
- RFC 0039: Identity store consolidation
- RFC 0041: Authorization model refresh

Feedback welcome — please leave comments inline or in the `#rfc-0042` channel. 🛠️
