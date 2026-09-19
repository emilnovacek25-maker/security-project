# Security Project — GitHub Policy Broker

Stav: **SPECIFIKAČNÍ FÁZE / IMPLEMENTACE BROKERU ZATÍM NEZAHAJOVAT**

Tento repozitář je autoritativní pracovní prostor pro formální bezpečnostní specifikaci GitHub Policy Brokeru.

## Nejvyšší bezpečnostní invariant

> Žádná GitHub write operace, která podle bezpečnostního modelu podléhá brokeru, nesmí být technicky proveditelná jinou cestou než prostřednictvím brokeru.

Současně platí:

```text
Co není explicitně povoleno
→ BLOCK

POLICY BLOCK
=
TECHNICKY NEMOŽNÁ OPERACE
```

## Cílová trust boundary

```text
USER
  ↓
ChatGPT
  │
  │ NO WRITE CREDENTIAL
  ↓
ACTION REQUEST
  ↓
POLICY / ACTION BROKER
  ↓
short-lived GitHub App credential
  ↓
GitHub
```

ChatGPT interpretuje požadavek, navrhuje operaci a připravuje obsah. Neautorizuje, není identity provider, nevydává credential a nemá přímý GitHub write credential.

Broker ověřuje identitu, canonicalizuje request, načítá autoritativní GitHub state, vyhodnocuje policy, vyžaduje případný CONFIRM, ověřuje potvrzený state, získává krátkodobý credential, provádí operaci, post-check a audit.

## Pořadí práce

1. `THREAT-MODEL.md` v0
2. `WRITE-PATHS.md` + credentials inventory
3. `THREAT-MODEL.md` v1
4. `ACTOR-PRINCIPALS.yaml` + `ACTOR-CAPABILITIES.yaml`
5. `CONTROLS.yaml` + `SECURITY-VIOLATIONS.yaml`
6. `CREDENTIAL-MODEL.md`
7. Action / Policy / Confirmation / Execution / Audit schémata + `CONFIRMATION-PROTOCOL.md`
8. Audit ownership + `DEPLOYMENT-SECURITY.md`
9. `INCIDENT-RESPONSE.md` + `RESPONSIBILITIES.md`
10. `MIGRATION.md` + `TEST-PLAN.md` + `OBSERVABILITY.md`
11. Review celé specifikace + Definition of Ready
12. Teprve poté implementace brokeru

## Stav

```text
KONCEPČNÍ OPONENTURA
→ UZAVŘENA

NORMATIVNÍ ZADÁNÍ
→ READY

SPECIFIKAČNÍ FÁZE
→ READY TO START

FORMÁLNÍ BEZPEČNOSTNÍ SPECIFIKACE
→ NOT COMPLETE

IMPLEMENTACE BROKERU
→ ZATÍM NEZAHAJOVAT
```
