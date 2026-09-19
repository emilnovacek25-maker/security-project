# THREAT-MODEL.md

Version: 0.1.0  
Status: DRAFT / THREAT MODEL v0  
Scope: GitHub Policy Broker security project

## 1. Účel

Tento dokument je počáteční threat model před konfrontací návrhu se skutečnými write cestami, principals, credentials a effective permissions v GitHub prostředí.

THREAT MODEL v0 je hypotéza. Po dokončení `WRITE-PATHS.md` a credentials inventory musí být aktualizován na THREAT MODEL v1.

## 2. Nejvyšší bezpečnostní invariant

```text
Žádná GitHub write operace,
která podle bezpečnostního modelu podléhá brokeru,
nesmí být technicky proveditelná jinou cestou
než prostřednictvím brokeru.

Co není explicitně povoleno
→ BLOCK

POLICY BLOCK
=
TECHNICKY NEMOŽNÁ OPERACE
```

Výjimkou mohou být pouze explicitně evidované, samostatně zabezpečené a schválené write cesty.

## 3. Assets

- **A001 GitHub repositories** — obsah, historie, branches/tags, PR, konfigurace a security-sensitive files.
- **A002 Security policy** — policy data, verze/digest, integrita pravidel a historie změn.
- **A003 Broker credentials** — GitHub App private key, installation tokeny, signing credentials a issuance flow.
- **A004 Human identity and approval** — identita oprávněného člověka, autentizovaný confirmation channel, nonce, single-use a approval digest.
- **A005 Audit evidence** — audit events, integrity, external anchor, retention a forenzní hodnota.
- **A006 Deployment environment** — broker host, runtime identity, deployment pipeline, configuration, secrets a backupy.
- **A007 CI / GitHub Actions context** — workflow permissions, privileged events, secrets, OIDC, third-party actions a reusable workflows.

## 4. Aktéři a principals

### Human
Zadává záměr a může potvrdit citlivou operaci. Jedna fyzická osoba může mít více logických rolí.

### ChatGPT
Interpretuje, navrhuje a připravuje obsah. Nesmí být identity provider a nesmí mít přímý GitHub write credential.

### Policy / Action Broker
Ověřuje identitu, canonicalizuje request, načítá autoritativní GitHub state, vyhodnocuje policy, ověřuje confirmation state, získává krátkodobý credential, provádí povolenou operaci, post-check a audit.

### GitHub App installation
Authentication principal brokera vůči GitHubu.

### GitHub Actions workflows/jobs
Samostatní principals. Každý privilegovaný workflow/job musí být inventarizován včetně triggeru, GITHUB_TOKEN permissions, secrets, OIDC, reusable workflows a externích credentials.

### Další potenciální principals
Dependabot, externí služby, lidské GitHub účty, API integrace a automatizované skripty. Jejich skutečná existence a capability musí být potvrzena inventurou.

## 5. Trust boundaries

- **TB001 Human → ChatGPT** — interpretační kanál; text může být neúplný, chybný nebo špatně interpretovaný.
- **TB002 ChatGPT → Broker** — ChatGPT není bezpečnostní autorita. Broker nevěří tvrzením LLM o identitě, potvrzení, GitHub state ani effective permissions.
- **TB003 Broker → GitHub** — privilegovaná write boundary; pouze po policy, state check a případném human confirmation.
- **TB004 GitHub Actions → GitHub / external services** — privilegovaná execution boundary; žádná privilegovaná write/external-auth capability bez explicitní výjimky.
- **TB005 Broker → Audit** — audit append-oriented; broker nesmí zpětně měnit historii.
- **TB006 Broker → Secret/signing service** — GitHub App private key je root credential a nesmí být vystaven ChatGPT, repozitáři, běžným workflow ani logům.

## 6. Hrozby v0

### T001 Prompt injection / compromised LLM behavior
**Scénář:** ChatGPT navrhne škodlivou operaci nebo se pokusí obejít policy.  
**Směr mitigace:** žádný write credential pro ChatGPT; broker nevěří LLM authority claims; policy nad nedůvěryhodným obsahem; default deny.

### T002 Direct write bypass outside broker
**Scénář:** jiný token, PAT, deploy key, App, Actions, external service, SSH credential nebo jiná API cesta provede změnu mimo broker.  
**Směr mitigace:** write-path a credential inventory; BROKERED / APPROVED_EXCEPTION / REMOVED; effective-permission verification; drift detection.

### T003 Policy evaluator bug
**Scénář:** broker vydá ALLOW tam, kde měl být BLOCK.  
**Směr mitigace:** deterministický evaluator, fail closed, unit/property/negative testy, control verification.

### T004 Policy integrity compromise
**Scénář:** policy je změněna bez oprávnění nebo broker načte neautorizovanou verzi.  
**Směr mitigace:** versioned policy, immutable digest/SHA, review, integrity checks, policy owner.

### T005 GitHub App private key compromise
**Scénář:** únik root credential aplikace.  
**Směr mitigace:** protected secret store/signing service, omezený přístup, rotation/revoke, emergency suspend/uninstall, incident response.

### T006 Installation token leakage
**Scénář:** krátkodobý token unikne během issuance nebo execution.  
**Směr mitigace:** krátká lifetime, repository/permission scope, no-log policy, revoke/validation.

### T007 Confirmation forgery
**Scénář:** ChatGPT nebo jiný klient tvrdí, že člověk operaci schválil.  
**Směr mitigace:** ChatGPT není identity provider; potvrzení přímo vůči brokeru přes autentizovaný kanál.

### T008 Confirmation replay
**Scénář:** dříve platné confirmation je použito znovu.  
**Směr mitigace:** unique request ID, nonce, single-use, expiry, replay detection, cancellation a concurrency rules.

### T009 TOCTOU / state drift after confirmation
**Scénář:** po potvrzení se změní base SHA, head SHA, diff/tree digest, policy digest nebo relevantní state.  
**Směr mitigace:** state-bound confirmation; bezprostřední re-check; mismatch => invalid.

### T010 Security-sensitive change disguised as normal update
**Scénář:** policy/workflow/credential/audit/confirmation/deployment změna je klasifikována jako běžný update.  
**Směr mitigace:** samostatné security-sensitive operation classes; AI direct modification default deny; human approval dle policy.

### T011 Privileged GitHub Actions workflow
**Scénář:** workflow získá effective permission v rozporu s capability modelem.  
**Směr mitigace:** workflow/job jako samostatný principal; declared vs effective comparison; mismatch => security violation.

### T012 pull_request_target misuse
**Scénář:** privilegovaný event spustí nedůvěryhodný PR kód nebo vystaví secrets/token.  
**Default posture:** DEFAULT DENY.  
**Výjimka:** security review, žádné spuštění nedůvěryhodného PR kódu, least privilege, omezené secrets, samostatný security test.

### T013 OIDC misuse via id-token: write
**Scénář:** workflow získá OIDC JWT a vymění jej za externí privilege.  
**Klasifikace:** PRIVILEGED EXTERNAL-AUTH CAPABILITY.  
**Default posture:** DEFAULT DENY.  
**Výjimka:** explicitní workflow/účel, audience/subject trust, minimální externí role, security test.

### T014 Supply-chain compromise
**Scénář:** kompromitovaná Python dependency, GitHub Action, CI runner, build/deploy pipeline, dependency confusion nebo malicious update.  
**Směr mitigace:** dependency locking, hash verification, immutable SHA pinning kritických Actions, dependency review, oddělení build/deploy identity a omezené CI permissions.

### T015 Compromised developer/admin workstation
**Scénář:** útočník získá lidské nebo deployment credentials přes kompromitovaný workstation.  
**Směr mitigace:** silná autentizace, minimalizace dlouhodobých credentials, role separation, incident response.

### T016 Social engineering / approval fatigue
**Scénář:** phishing, fake confirmation UI, pretexting, impersonation, approval fatigue nebo confirmation confusion.  
**Směr mitigace:** jasné confirmation UX se zobrazením repo/action/base/head/security-sensitive paths/risk/expiry; autentizace dle threat modelu.

### T017 Broker host compromise
**Scénář:** útočník ovládne host brokera.  
**Směr mitigace:** deployment hardening, secret isolation, restricted admin access, patching, disk/backup protection, emergency suspend, independent audit.

### T018 Audit tampering or omission
**Scénář:** broker přepíše/smaže historii nebo nezaloguje škodlivou operaci.  
**Směr mitigace:** append-oriented store, broker bez UPDATE/DELETE, tamper-evidence, external anchor a oddělený ownership.

### T019 Broker / policy / audit unavailability
**Scénář:** broker, policy nebo audit není dostupný.  
**Normativně:** policy unavailable => BLOCK WRITE; audit unavailable => BLOCK WRITE; identity uncertain => BLOCK.  
Availability model musí rozhodnout STRICT FAIL-CLOSED vs CONTROLLED EMERGENCY ACCESS.

### T020 Emergency bypass becomes permanent bypass
**Scénář:** break-glass obejde hlavní invariant.  
**Směr mitigace:** nevytvářet automaticky; pokud existuje, APPROVED_EXCEPTION, silná autentizace, explicitní aktivace, time-bound scope, separate audit, auto-expiry a post-incident review.

### T021 Security drift
**Scénář:** přibude PAT/App/deploy key/write workflow nebo se změní permissions/deployment.  
**Směr mitigace:** bottom-up validation a pravidelné porovnání actual vs documented state; mismatch => SECURITY VIOLATION.

### T022 Repository identity confusion
**Scénář:** repo je přejmenováno/transferováno nebo je použit podobný název.  
**Směr mitigace:** repository_id jako primární identity; owner/name jako display/consistency metadata; mismatch => BLOCK.

## 7. Default fail-closed baseline

```text
policy unavailable            → BLOCK WRITE
audit unavailable             → BLOCK WRITE
identity uncertain            → BLOCK
repository identity mismatch  → BLOCK
state changed                 → BLOCK
unknown action                → BLOCK
unknown repository            → BLOCK
unknown write path            → BLOCK
```

Zakázaný fallback:

```text
broker nefunguje
→ použij GitHub přímo
```

## 8. Security violations v0

```text
V001 Declared capability mismatch
V002 Unknown write path
V003 Write outside broker without approved exception
V004 Unexpected credential
V005 Security-sensitive modification outside approval flow
V006 Confirmation state mismatch
V007 Policy integrity failure
V008 Audit integrity failure
V009 Security drift detected
```

Detailní response budou definovány v `SECURITY-VIOLATIONS.yaml`.

## 9. Residual risks v0

- oprávněný člověk může schválit škodlivou operaci,
- kompromitovaný host může poškodit execution flow,
- kompromitovaný GitHub účet může měnit repository-level configuration,
- GitHub je trusted external dependency,
- neznámé write cesty mohou existovat do dokončení inventury,
- availability/emergency-access model není ještě uzavřen,
- přesný authentication/session/device-binding model confirmation není ještě uzavřen.

## 10. Povinné vstupy pro v1

THREAT MODEL v1 nesmí vzniknout bez:
- `WRITE-PATHS.md`,
- credentials inventory bez secretů,
- GitHub Actions principal inventory,
- effective permissions inventory,
- skutečných repository identities,
- rozhodnutého nebo skutečného deployment modelu.

## 11. Exit criteria pro THREAT MODEL v0

THREAT MODEL v0 je připraven pro další krok, pokud:
- jsou definovány assets,
- jsou definováni základní actors/principals,
- jsou definovány hlavní trust boundaries,
- jsou evidovány počáteční hrozby,
- je definován fail-closed baseline,
- je jasně označeno, co musí potvrdit reálná inventura.

Další krok: `WRITE-PATHS.md` + credentials inventory.
