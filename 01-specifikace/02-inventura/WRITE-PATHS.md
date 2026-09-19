# WRITE-PATHS.md

Version: 0.1.0  
Status: INVENTORY TEMPLATE / NOT YET POPULATED

## Pravidlo

Každá GitHub write cesta musí být identifikována podle skutečného principalu, credentialu a effective permission.

Cílová klasifikace:

- `BROKERED`
- `APPROVED_EXCEPTION`
- `REMOVED`

`UNKNOWN` je pouze dočasný stav inventury.

## Records

| ID | Actor | Principal | Credential type | Target | Operation | Effective permission | Current state | Target classification | Evidence | Threats |
|---|---|---|---|---|---|---|---|---|---|---|

## Povinné kategorie ke kontrole

- human GitHub UI/admin,
- git SSH,
- git HTTPS,
- PAT,
- GitHub App,
- OAuth/external service,
- GitHub Actions `GITHUB_TOKEN`,
- Actions OIDC/external auth,
- deploy key,
- REST API,
- GraphQL mutation,
- Git Database API,
- PR merge,
- ref/branch/tag update,
- settings/ruleset/admin mutation.

## Migration rule

Pro každou existující cestu:

```text
CURRENT STATE
↓
TARGET STATE
↓
MIGRATION
↓
VERIFICATION
↓
OLD PATH DISABLED
```

Klíčový test není pouze „nová cesta funguje“, ale také „stará neautorizovaná cesta už nefunguje“.
