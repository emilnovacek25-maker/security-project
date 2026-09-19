# CREDENTIALS.md

Version: 0.1.0  
Status: INVENTORY TEMPLATE / NO SECRET VALUES

## Bezpečnostní pravidlo

Tento soubor nesmí obsahovat:
- token values,
- private keys,
- passwords,
- session cookies,
- recovery codes,
- raw signing material.

## Records

| ID | Type | Principal/owner | Purpose | Scope | Effective permissions | Storage class | Exportable | Expiry | Rotation owner | Revoke method | Related write paths | Status | Evidence |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|

## Minimální typy k prověření

- GitHub App private key/signing capability,
- GitHub App installation token,
- fine-grained PAT,
- classic PAT,
- SSH key,
- deploy key,
- OAuth credential,
- `GITHUB_TOKEN`,
- OIDC trust/exchanged credential,
- external CI/CD credential,
- confirmation/session secret,
- audit-anchor credential.
