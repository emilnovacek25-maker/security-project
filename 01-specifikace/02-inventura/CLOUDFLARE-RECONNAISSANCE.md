# CLOUDFLARE-RECONNAISSANCE.md

Status: CHECKLIST / NOT YET EXECUTED  
Phase: 01-specifikace/02-inventura  
Reference: REF001 — cloudflare/security-audit-skill

## 1. Účel

Tento krok používá Cloudflare security-audit pouze jako externí metodickou referenci pro reconnaissance a coverage-led hunting.

Není to autoritativní inventura GitHub effective state.

Výstupem není tvrzení „toto jsou všechny write cesty“, ale:
- coverage plan,
- seznam oblastí, které musí naše autoritativní inventura ověřit,
- source/local-state findings,
- případné `needs_validation` položky.

## 2. Povinné metadata každého běhu

Zaznamenat:

- repository: `cloudflare/security-audit-skill`
- source commit / immutable version: **REQUIRED**
- date: **REQUIRED**
- audit profile: **REQUIRED**
- target scope: **REQUIRED**
- operator: **REQUIRED**
- sandbox status: **REQUIRED**
- limitations: **REQUIRED**
- resulting report/coverage artifact path: **REQUIRED**

Nikdy nepoužívat pouze pohyblivý údaj typu `@main` jako historickou identitu metodiky.

## 3. Instalace reference

Referenční instalační příkaz:

```bash
npx skills add https://github.com/cloudflare/security-audit-skill \
  --skill security-audit
```

Před použitím zaznamenat přesný upstream commit/version použitý pro daný běh.

## 4. Reconnaissance scope

Prověřit source/local-state oblasti relevantní pro broker zejména:

- authentication/authorization code,
- canonicalization/parsing,
- tool/action dispatch,
- confirmation/approval binding,
- retries/resume/idempotency,
- secret handling,
- logging/audit serialization,
- GitHub API adapters,
- policy loading/evaluation,
- CI workflows,
- third-party Actions,
- dependency manifests/lockfiles,
- deployment manifests,
- configuration files,
- tests zaměřené na authorization/security boundaries.

## 5. Explicitní hranice

Cloudflare-style reconnaissance NESMÍ být zaměněno za naši autoritativní GitHub inventuru.

Reconnaissance samo nepotvrzuje:

- aktuální GitHub Apps,
- App installation IDs,
- skutečné App permissions,
- PATs,
- OAuth Apps,
- deploy keys,
- SSH keys uživatelů,
- repo IDs,
- team/user effective permissions,
- branch/ruleset effective enforcement,
- workflow effective permissions odvozené z live GitHub configuration,
- externí identity/credentials mimo dostupný source/local-state.

Tyto položky patří do `AUTHORITATIVE-INVENTORY-CHECKLIST.md`.

## 6. Sandbox gate

Pokud validační krok vyžaduje spuštění nedůvěryhodného nebo analyzovaného kódu, předpokládaný profil je:

- OS-enforced isolation,
- no external network, pokud není explicitně povolen konkrétní endpoint,
- empty allowlisted environment,
- target read-only,
- writes pouze do scratch prostoru,
- explicit resource limits.

Pokud sandbox není dostupný:

```text
DO NOT RUN RISKY VALIDATION
↓
finding = needs_validation
↓
record reason
```

Žádné improvizované spouštění na produkčním broker hostu.

## 7. Coverage questions

Reconnaissance má vytvořit otázky minimálně pro:

- T001 prompt injection,
- T002 direct write bypass,
- T007–T009 confirmation/action binding,
- T011–T013 privileged GitHub Actions/OIDC,
- T014A–D supply chain,
- T017 broker host,
- T018A–D audit integrity,
- T023 human admin bypass,
- T024 bootstrap,
- T025 forensic evidence,
- T026 canonicalization/parser mismatch,
- T027 Action Request replay/idempotency,
- T028 secret disclosure.

## 8. Povinný výstup

Po běhu musí existovat coverage record s položkami:

```text
AREA
SOURCE/EVIDENCE
STATUS
THREAT IDS
QUESTIONS FOR LIVE INVENTORY
FINDINGS
NEEDS_VALIDATION
LIMITATIONS
```

Allowed statuses:

- COVERED_SOURCE
- PARTIAL
- NOT_FOUND
- NEEDS_LIVE_INVENTORY
- NEEDS_VALIDATION
- OUT_OF_SCOPE_WITH_REASON

## 9. Exit criteria

Reconnaissance je dokončeno, když:

- použitá reference je pinovaná na konkrétní commit/version,
- scope je zaznamenaný,
- všechny relevantní oblasti mají coverage status,
- live-state otázky jsou explicitně předány autoritativní inventuře,
- neexistuje tvrzení o GitHub effective state založené pouze na source reconnaissance.
