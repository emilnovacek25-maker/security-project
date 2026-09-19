# THREAT-MODEL.md

Version: 0.4.0  
Status: DRAFT / THREAT MODEL v0  
Scope: GitHub Policy Broker security project

## 1. Účel

Tento dokument je formální počáteční threat model projektu GitHub Policy Broker před konfrontací návrhu se skutečnými write cestami, principals, credentials, effective permissions a deploymentem.

`THREAT MODEL v0` je řízená hypotéza. Nesmí být považován za popis skutečného stavu GitHub prostředí. Jeho účelem je:

- vymezit scope a non-goals,
- definovat security objectives a invariants,
- popsat assets,
- rozlišit actors, threat actors, principals, credentials a effective permissions,
- definovat trust assumptions a Trusted Computing Base,
- popsat data flows a trust boundaries,
- vytvořit katalog hrozeb s attack path, impact, candidate controls, enforcement, detection, test/evidence a residual risk,
- určit otevřená rozhodnutí,
- stanovit, co musí potvrdit reálná inventura.

Po dokončení `WRITE-PATHS.md`, credentials inventory, principal inventory a effective-permission inventory bude vytvořen `THREAT MODEL v1`.

---

## 2. Scope

### 2.1 In scope

Threat model pokrývá zejména:

- GitHub write operations,
- GitHub repositories spravované brokerem,
- Policy / Action Broker,
- policy data a policy evaluator,
- GitHub App a její credentials,
- installation tokeny,
- human confirmation flow,
- GitHub Actions privilege model,
- CI/CD trust,
- supply chain,
- audit a external anchor,
- deployment host a runtime,
- incident response,
- availability a emergency-access model,
- security drift,
- bootstrap/root-of-trust operace.

### 2.2 Out of scope pro v0

V této fázi nejsou detailně modelovány:

- interní implementace GitHub platformy,
- fyzická bezpečnost GitHub datacenter,
- obecná endpoint security všech uživatelských zařízení mimo vazbu na broker credentials,
- obecný malware model nesouvisející s brokerem nebo jeho credentials,
- právní/regulatorní compliance nad rámec dat, která broker skutečně ukládá,
- detailní produkční síťová topologie, dokud není rozhodnut deployment model.

Out-of-scope neznamená ignorováno. Pokud inventura nebo deployment design ukáže, že je některý bod bezpečnostně relevantní, musí být převeden do scope v1.

### 2.3 Explicitní non-goals

Tento threat model nemá za cíl:

- chránit proti fyzickému útoku na GitHub datacentra,
- modelovat interní zero-day zranitelnosti samotné GitHub platformy,
- garantovat bezpečnost proti útočníkovi s neomezenými prostředky,
- řešit kompromitaci samotného modelu/provozovatele ChatGPT mimo rozhraní a capabilities, které tento systém skutečně používá,
- nahrazovat obecný endpoint-security program pro všechna lidská zařízení,
- nahrazovat bezpečnostní model TLS/DNS/GitHub jako externích trusted dependencies,
- dokazovat právní/regulatorní compliance, dokud nejsou známa skutečná auditní data a provozní kontext.

Pokud některý non-goal začne přímo ovlivňovat bezpečnostní invariant systému, musí být v další verzi přeřazen do scope.

---

## 3. Security objectives a invariants

### I001 — Single brokered AI write path

```text
Žádná GitHub write operace,
která podle bezpečnostního modelu podléhá brokeru,
nesmí být technicky proveditelná jinou cestou
než prostřednictvím brokeru.
```

### I002 — Default deny

```text
Co není explicitně povoleno
→ BLOCK
```

### I003 — Technical meaning of BLOCK

Pro principal a write path podléhající policy brokeru:

```text
POLICY BLOCK
=
operace technicky neproveditelná
touto ani jinou neautorizovanou cestou
```

Tento invariant se nevztahuje na explicitně evidované `APPROVED_EXCEPTION` / root-governance cesty; ty musí mít vlastní controls, audit a governance.

### I004 — ChatGPT has no write credential

ChatGPT nesmí držet ani získat GitHub credential umožňující přímý write.

### I005 — ChatGPT is not an identity provider

ChatGPT nesmí být autoritou pro tvrzení:

- kdo je uživatel,
- zda uživatel operaci schválil,
- jaký je aktuální GitHub state,
- jaké jsou effective permissions.

### I006 — Broker fetches authoritative state

Broker musí bezpečnostně relevantní GitHub state načítat sám z autoritativního zdroje.

### I007 — Confirmation is state-bound

```text
CONFIRMED STATE != CURRENT STATE
→ CONFIRMATION INVALID
```

### I008 — Unknown means deny

Unknown action, unknown repository, unknown credential, unknown write path nebo nejistá identita nesmí být interpretovány jako allow.

### I009 — Declared capability must equal effective permission

```text
DECLARED CAPABILITY != EFFECTIVE PERMISSION
→ SECURITY VIOLATION
```

### I010 — Audit history survives broker compromise

```text
COMPROMISE BROKERU
≠
MOŽNOST PŘEPSAT AUDITNÍ HISTORII
```

### I011 — No direct fallback

```text
broker unavailable
→ write unavailable
```

Nesmí existovat automatický fallback na přímý GitHub write.

### I012 — Security-sensitive changes are separate classes

Změny policy, workflows, credential configuration, audit, confirmation a deployment security nesmí být klasifikovány jako běžný file update.

### I013 — Human privileged paths are explicit

Lidské admin/owner cesty nejsou skrytou výjimkou. Musí být klasifikovány jako:

```text
BROKERED
APPROVED_EXCEPTION
REMOVED
```

---

## 4. Assets

### A001 — GitHub repositories
Chráníme:
- obsah souborů,
- commit history,
- branches a tags,
- pull requests,
- repository settings,
- branch/ruleset konfiguraci,
- security-sensitive files.

### A002 — Security policy
Chráníme:
- policy data,
- policy schema,
- policy version/digest,
- integritu pravidel,
- historii změn,
- aktivní policy pointer.

### A003 — Broker credentials
Chráníme:
- GitHub App private key,
- signing capability,
- installation tokeny,
- credential issuance flow,
- případné deployment credentials.

### A004 — Human identity and approval
Chráníme:
- autentizovanou lidskou identitu,
- confirmation session,
- request ID,
- nonce,
- single-use state,
- approval digest,
- expiry.

### A005 — Audit evidence
Chráníme:
- audit events,
- order events,
- timestamps,
- integrity chain,
- external anchor,
- retention,
- forensic usability.

### A006 — Broker software
Chráníme:
- policy evaluator,
- canonicalization logic,
- GitHub adapter,
- confirmation logic,
- execution logic,
- audit client.

### A007 — Deployment environment
Chráníme:
- host,
- runtime identity,
- network exposure,
- OS,
- backups,
- deployment pipeline,
- secret store,
- configuration.

### A008 — GitHub Actions / CI context
Chráníme:
- workflow definitions,
- job permissions,
- GITHUB_TOKEN,
- secrets,
- OIDC,
- third-party actions,
- reusable workflows,
- runners.

### A009 — Operational availability
Chráníme:
- ability to make authorized writes,
- ability to verify state,
- ability to audit,
- ability to revoke access.

---

## 5. Actors a logical roles

### ACT001 — Human requester
Zadává záměr.

### ACT002 — Human approver
Provádí explicitní potvrzení citlivé operace.

### ACT003 — Security owner
Schvaluje bezpečnostní pravidla, výjimky a release.

### ACT004 — ChatGPT
Interpretuje, navrhuje a připravuje obsah.

### ACT005 — Policy / Action Broker
Rozhoduje a vynucuje brokered write flow.

### ACT006 — GitHub
Autoritativní platforma a vykonavatel API operací.

### ACT007 — GitHub Actions
Logický actor zahrnující jednotlivé workflow/job principals.

### ACT008 — Dependabot
Samostatný automatizační actor, pokud je v prostředí použit.

### ACT009 — External service
Jakákoli třetí služba s API, OAuth, GitHub App, SSH nebo tokenovým přístupem.

### ACT010 — Human GitHub admin/owner
Privilegovaný lidský actor schopný měnit repository nebo App konfiguraci.

---

## 6. Authentication principals, credentials a effective permissions

Bezpečnostní model rozlišuje:

```text
LOGICAL ACTOR
      ↓
AUTHENTICATION PRINCIPAL
      ↓
CREDENTIAL
      ↓
EFFECTIVE PERMISSION
```

Příklady:

```text
github_actions
↓
workflow/job
↓
GITHUB_TOKEN / OIDC / secret
↓
effective permissions
```

```text
broker
↓
GitHub App installation
↓
installation token
↓
contents:write / pull_requests:write / ...
```

```text
human_admin
↓
GitHub user account
↓
session / SSH key / PAT
↓
owner/admin/write capability
```

Každý principal a credential musí být v inventuře. Credential inventory nesmí obsahovat secret value.

---

## 7. Threat actors a attacker capabilities

### TA001 — External unauthenticated attacker
Předpokládané schopnosti:
- internet access,
- pokusy o probing/phishing,
- přístup k veřejným datům.

Nepředpokládáme:
- legitimní GitHub credential,
- interní host access.

### TA002 — Prompt-injection content attacker
Schopnosti:
- ovlivnit obsah Issue/PR/README/SKILL/web content,
- vložit instrukce určené LLM.

Cíl:
- přimět ChatGPT k návrhu škodlivé operace nebo obcházení pravidel.

### TA003 — Compromised human GitHub account
Schopnosti:
- vše, co dovolí skutečné account permissions,
- případně změnit App/repo/security configuration.

### TA004 — Compromised developer workstation
Schopnosti:
- odcizit lokální credentials,
- manipulovat zdrojovým kódem,
- manipulovat deploy procesem podle skutečných práv uživatele.

### TA005 — Compromised broker host
Schopnosti:
- spouštět kód jako broker principal,
- číst lokální runtime data,
- pokusit se zneužít signing/token issuance.

### TA006 — Malicious/compromised dependency
Schopnosti:
- spustit kód během build/runtime podle kontextu dependency.

### TA007 — Compromised GitHub Action / runner
Schopnosti:
- využít effective workflow permissions,
- číst dostupné secrets,
- získat OIDC JWT, pokud je povolen.

### TA008 — Malicious or mistaken privileged insider
Schopnosti:
- využít legitimní lidské pravomoci,
- potvrdit chybnou operaci,
- změnit konfiguraci.

### TA009 — Compromised external service
Schopnosti:
- využít OAuth/App/token permissions, které daná integrace skutečně má.

---

## 8. Trust assumptions

### AS001 — GitHub platform
GitHub je trusted external dependency pro repository state, identity primitives a API enforcement.

### AS002 — TLS / transport security
Transportní integrita a autenticita GitHub a confirmation komunikace jsou předpokládány.

### AS003 — Host OS
Host OS brokera je součást TCB.

### AS004 — Secret/signing service
Pokud se použije vault/signing service, jeho integrita a access control jsou trusted external/internal controls.

### AS005 — Human security owner
Security owner je trusted authority pro explicitní bezpečnostní rozhodnutí a výjimky.

### AS006 — Time source
Správný čas je bezpečnostně relevantní pro expiry, replay a audit chronology.

### AS007 — Audit anchor
External anchor musí být spravován odděleným credentialem nebo trust domain tak, aby broker nemohl zpětně měnit historii.

### AS008 — GitHub repository identity
`repository_id` je bezpečnostní identita; owner/name slouží jako display/consistency metadata.

### AS009 — Sandbox availability for external audit
Pro plný externí security audit, který by potřeboval spouštět analyzovaný kód nebo validační kroky, se předpokládá OS-enforced sandbox s minimálně:
- bez externí sítě, pokud není konkrétní krok výslovně povolen,
- prázdným allowlisted environmentem,
- read-only targetem,
- scratch-only writes,
- explicitními resource limits.

Pokud vhodný sandbox není k dispozici:
- rizikový validační krok se nespouští,
- nález zůstane `needs_validation`,
- nesmí být povýšen na `confirmed` pouze na základě nebezpečné improvizované validace,
- důvod neprovedené validace se zaznamená.

Pokud některý assumption selže, musí být vyhodnocen odpovídající incident nebo security violation.

---

## 9. Trusted Computing Base (TCB)

Minimální komponenty, jejichž kompromitace může porušit hlavní bezpečnostní invariant:

- policy evaluator,
- request canonicalizer,
- enforcement/execution component,
- human confirmation service,
- credential/signing service,
- GitHub App permission configuration,
- broker runtime/host,
- active policy selection mechanism,
- audit anchor mechanism,
- security-owner authentication path.

Cíl implementace je TCB minimalizovat. Komponenta nemá být v TCB jen proto, že je součást systému.

**Terminologie:** tato sekce popisuje `local TCB` systému, který sami provozujeme. GitHub platforma není součástí local TCB; je vedena jako **external trusted computing dependency**. Její kompromitace může bezpečnostní invariant porušit, ale není lokálně kontrolovatelná stejným mechanismem jako broker, host, policy evaluator nebo signing service.

---

## 10. Data flows a trust boundaries

### TB001 — Human → ChatGPT
**Data:** záměr, text úkolu, případné soubory.  
**Trust:** untrusted interpreted input.  
**Broker nesmí od ChatGPT převzít jako autoritativní:** identity, confirmation, state, permissions.  
**Failure behavior:** chybná interpretace nesmí sama umožnit write.

### TB002 — ChatGPT → Broker
**Data:** proposed action, proposed content, repository hint, operation intent.  
**Trust:** untrusted request proposal.  
**Authentication:** nesmí být odvozena jen z textu LLM.  
**Integrity requirement:** broker canonicalizuje request a dohledává state sám.  
**Failure behavior:** ambiguity/unknown → BLOCK.

### TB003 — Broker → GitHub
**Data:** autorizovaný API request.  
**Authentication:** short-lived scoped GitHub App credential.  
**Integrity:** request musí odpovídat policy-approved state transition.  
**Failure behavior:** mismatch/error → BLOCK/FAIL CLOSED.

### TB004 — GitHub → Broker
**Data:** repository identity, refs, SHAs, diff, PR state, CI state, API response.  
**Trust:** authoritative external state within GitHub trust assumption.  
**Failure behavior:** missing/ambiguous/stale data → no privileged execution.

### TB005 — Broker → Confirmation service / Human
**Data:** action summary, repo ID, PR, base/head refs+SHAs, diff/tree digest, policy digest, request digest, expiry, nonce.  
**Trust:** confirmation UI must present exact state.  
**Failure behavior:** expired/replayed/changed state → INVALID.

### TB006 — Broker → Credential/signing service
**Data:** signing/token request.  
**Trust:** privileged boundary.  
**Requirement:** private key not exposed to ChatGPT/repo/workflows/logs.

### TB007 — Broker → Audit store
**Data:** security event, decision, state, result.  
**Requirement:** append-oriented; broker no UPDATE/DELETE.  
**Failure behavior:** audit unavailable → BLOCK WRITE.

### TB008 — Audit store → Anchor store
**Data:** current audit head hash / checkpoint.  
**Requirement:** independent credential and trust domain.

### TB009 — GitHub Actions → GitHub
**Data:** workflow API operations.  
**Trust:** each workflow/job is a separate principal.  
**Default posture:** no privileged write capability without approved exception.

### TB010 — GitHub Actions → External identity provider
**Data:** OIDC JWT or exchanged credentials.  
**Default posture:** `id-token: write` = privileged external-auth capability, default deny.

---

## 11. Threat catalogue v0

Každá hrozba obsahuje:
- Asset,
- Threat actor,
- Preconditions,
- Attack path,
- Impact,
- Candidate controls,
- Enforcement,
- Detection,
- Test/evidence,
- Residual risk,
- Risk status.

### T001 — Prompt injection / compromised LLM behavior
**Assets:** A001, A002, A004.  
**Threat actor:** TA002.  
**Preconditions:** ChatGPT zpracuje nedůvěryhodný obsah.  
**Attack path:** malicious content → LLM instruction override attempt → harmful proposed action.  
**Impact:** pokus o neautorizovaný write nebo security-sensitive change.  
**Candidate controls:** C001 no ChatGPT write credential; C002 broker authoritative-state lookup; C018 untrusted-content/instruction hierarchy.  
**Enforcement:** credential isolation + broker policy.  
**Detection:** denied anomalous requests, prompt-injection test cases.  
**Test/evidence:** pokus přimět ChatGPT k direct write musí technicky selhat.  
**Residual risk:** člověk může škodlivý návrh omylem potvrdit.  
**Risk status:** HIGH / likelihood UNKNOWN pending inventory.

### T002 — Direct write bypass outside broker
**Assets:** A001, A002.  
**Threat actor:** TA003, TA004, TA007, TA009.  
**Preconditions:** existuje credential/write path mimo broker.  
**Attack path:** alternate credential/path → GitHub → state change without broker decision.  
**Impact:** ztráta významu POLICY BLOCK.  
**Candidate controls:** write-path inventory; credential inventory; remove/restrict alternate paths; rulesets where available; drift detection.  
**Enforcement:** GitHub permission configuration + credential removal + broker-only AI access.  
**Detection:** periodic bottom-up inventory and GitHub state comparison.  
**Test/evidence:** každá non-approved legacy path musí po migraci technicky FAIL.  
**Residual risk:** human owner/admin může zůstat explicitní approved exception.  
**Risk status:** CRITICAL / likelihood UNKNOWN pending inventory.

### T003 — Policy evaluator bug
**Assets:** A001, A002, A006.  
**Threat actor:** non-malicious defect / TA006 if code compromised.  
**Preconditions:** logic bug or malformed policy input.  
**Attack path:** invalid evaluation → false ALLOW.  
**Impact:** neautorizovaná operace.  
**Candidate controls:** deterministic evaluator, schemas, negative tests, property tests, fail closed.  
**Enforcement:** evaluator runtime.  
**Detection:** tests, audit anomalies, differential/replay testing.  
**Test/evidence:** malformed/unknown/conflicting policy → BLOCK.  
**Residual risk:** unknown implementation defects.  
**Risk status:** CRITICAL.

### T004 — Policy integrity compromise
**Assets:** A002.  
**Threat actor:** TA003, TA004, TA005, TA006.  
**Preconditions:** access to policy source or active-policy selection.  
**Attack path:** unauthorized policy change → broker loads permissive policy.  
**Impact:** systemic authorization bypass.  
**Candidate controls:** immutable policy digest/SHA, review, separate policy owner, integrity validation.  
**Enforcement:** policy loader + repository/process governance.  
**Detection:** policy digest mismatch, unexpected policy change.  
**Test/evidence:** unauthorized policy state must cause BLOCK.  
**Residual risk:** authorized security owner may approve flawed policy.  
**Risk status:** CRITICAL.

### T005 — GitHub App private key compromise
**Assets:** A003.  
**Threat actor:** TA004, TA005, TA006.  
**Preconditions:** access to root App credential.  
**Attack path:** key theft → JWT → installation token issuance.  
**Impact:** bypass broker runtime controls within App permission scope.  
**Candidate controls:** vault/sign-only service, non-exportability where possible, rotation/revoke, minimal App permissions.  
**Enforcement:** secret/signing infrastructure.  
**Detection:** unusual token issuance, host compromise indicators.  
**Test/evidence:** key must not exist in repo, Actions secrets, logs or ChatGPT context.  
**Residual risk:** signing-service compromise.  
**Risk status:** CRITICAL.

### T006 — Installation token leakage
**Assets:** A003, A001.  
**Threat actor:** TA004, TA005, TA006.  
**Attack path:** token exposed in log/process/memory/transport → direct GitHub API use.  
**Impact:** temporary write bypass within token scope.  
**Candidate controls:** short lifetime, minimum permissions/repositories, no-log rule, process isolation.  
**Detection:** audit/API anomalies.  
**Test/evidence:** logs and error paths must redact token.  
**Residual risk:** token remains usable until expiry/revocation.  
**Risk status:** HIGH.

### T007 — Confirmation forgery
**Assets:** A004, A001.  
**Threat actor:** TA002, TA009.  
**Attack path:** client/LLM sends `confirmed=true` without human action.  
**Impact:** sensitive operation executes without human approval.  
**Candidate controls:** authenticated confirmation channel; ChatGPT cannot attest approval.  
**Enforcement:** confirmation service.  
**Detection:** confirmation record absent/invalid.  
**Test/evidence:** LLM-provided confirmation flag alone → BLOCK.  
**Residual risk:** compromised human account.  
**Risk status:** CRITICAL.

### T008 — Confirmation replay
**Assets:** A004.  
**Threat actor:** TA003, TA009.  
**Attack path:** capture valid approval → reuse later.  
**Impact:** duplicate or unintended execution.  
**Candidate controls:** request ID, nonce, single-use, expiry, replay registry, cancellation.  
**Enforcement:** confirmation service/data store.  
**Detection:** replay attempt event.  
**Test/evidence:** second use of same confirmation → BLOCK + audit event.  
**Residual risk:** concurrency race if atomic consume fails.  
**Risk status:** HIGH.

### T009 — TOCTOU / state change after confirmation
**Assets:** A001, A004.  
**Threat actor:** TA003 or normal concurrent change.  
**Attack path:** approval on state A → base/head/policy changes → execution on state B.  
**Impact:** user schválí jinou operaci než se provede.  
**Candidate controls:** bind base_sha, head_sha, diff/tree digest, policy digest, request digest, merge method; immediate re-check.  
**Enforcement:** broker before execution.  
**Detection:** state mismatch.  
**Test/evidence:** mutate base/head after approval → confirmation invalid.  
**Residual risk:** API semantics/race not covered by local checks.  
**Risk status:** CRITICAL.

### T010 — Security-sensitive change disguised as ordinary update
**Assets:** A002, A003, A005, A006, A007.  
**Attack path:** modify sensitive path through generic update classification.  
**Impact:** change enforcement or credentials without required approval.  
**Candidate controls:** dedicated operation classes; protected path classification.  
**Enforcement:** canonicalizer + policy.  
**Detection:** path/category mismatch.  
**Test/evidence:** generic UPDATE_FILE against sensitive path → DENY or elevated flow.  
**Residual risk:** sensitive file not classified.  
**Risk status:** HIGH.

### T011 — Privileged GitHub Actions workflow
**Assets:** A001, A007.  
**Threat actor:** TA007.  
**Attack path:** workflow/job receives write permission not declared in model.  
**Impact:** parallel write path outside broker.  
**Candidate controls:** per-workflow principal inventory; declared/effective comparison; least privilege.  
**Enforcement:** workflow permissions + repository configuration.  
**Detection:** drift scan.  
**Test/evidence:** effective write permission without approved declaration → SECURITY VIOLATION.  
**Residual risk:** GitHub behavior/configuration changes.  
**Risk status:** CRITICAL until inventory complete.

### T012 — `pull_request_target` misuse
**Assets:** A007, A003, A001.  
**Threat actor:** TA007 / malicious PR author.  
**Attack path:** privileged base-repo context → checkout/run untrusted PR code → token/secrets abuse.  
**Impact:** credential theft or write bypass.  
**Candidate controls:** DEFAULT DENY; security-reviewed exception only.  
**Enforcement:** workflow policy/lint + review.  
**Detection:** workflow scan.  
**Test/evidence:** unauthorized use → fail security validation.  
**Residual risk:** reviewed workflow may still contain logic flaw.  
**Risk status:** HIGH.

### T013 — OIDC misuse via `id-token: write`
**Assets:** A007, external resources.  
**Threat actor:** TA007.  
**Attack path:** workflow obtains OIDC JWT → exchanges for external credential.  
**Impact:** privilege escalation outside GitHub.  
**Candidate controls:** DEFAULT DENY; explicit subject/audience trust; minimum external role.  
**Enforcement:** GitHub workflow permissions + external IdP policy.  
**Detection:** inventory and external credential logs.  
**Test/evidence:** unauthorized workflow cannot mint usable external credential.  
**Residual risk:** external IdP misconfiguration.  
**Risk status:** HIGH.

### T014A — Malicious Python dependency
**Assets:** A006, A003.  
**Threat actor:** TA006.  
**Attack path:** malicious package executes during build/runtime.  
**Impact:** broker or credential compromise.  
**Controls:** lockfile, hashes where feasible, review, minimal dependency set.  
**Test/evidence:** dependency integrity checks.  
**Residual risk:** trusted package takeover.  
**Risk status:** HIGH.

### T014B — Dependency confusion / malicious update
**Assets:** A006.  
**Attack path:** resolver selects attacker-controlled package/version.  
**Controls:** explicit indexes, pinning, provenance checks.  
**Risk status:** HIGH.

### T014C — Compromised GitHub Action / mutable tag
**Assets:** A007.  
**Attack path:** third-party Action ref changes or upstream compromised.  
**Controls:** critical Actions pinned to immutable commit SHA, allowlist/review.  
**Risk status:** HIGH.

### T014D — Compromised CI runner / build pipeline
**Assets:** A006, A007, A003.  
**Attack path:** runner executes attacker-controlled code with secrets/permissions.  
**Controls:** runner isolation, minimal secrets, separate build/deploy identity.  
**Risk status:** HIGH.

### T015 — Compromised developer/admin workstation
**Assets:** A003, A006, A001.  
**Threat actor:** TA004.  
**Attack path:** steal session/PAT/SSH/deploy credentials or alter code before review.  
**Candidate controls:** strong auth, minimal long-lived credentials, role separation, review, incident response.  
**Residual risk:** privileged human endpoint remains high-value target.  
**Risk status:** HIGH.

### T016 — Social engineering / approval fatigue
**Assets:** A004.  
**Threat actor:** TA001, TA002.  
**Attack path:** phishing/fake UI/pretexting/repeated prompts → human approves harmful action.  
**Candidate controls:** explicit confirmation UX showing repo, action, PR, base/head, sensitive paths, risk class, expiry; optional phishing-resistant auth based on v1 threat decision.  
**Test/evidence:** confirmation UX scenarios.  
**Residual risk:** human judgment can still fail.  
**Risk status:** HIGH.

### T017 — Broker host compromise
**Assets:** A003, A005, A006.  
**Threat actor:** TA005.  
**Attack path:** host access → alter runtime, steal token, suppress local logs.  
**Impact:** broker-mediated write abuse.  
**Candidate controls:** hardened host, least-privilege service account, patching, restricted SSH/admin, disk/backup protection, secret isolation, emergency App suspend.  
**Detection:** host monitoring + independent audit.  
**Residual risk:** active compromise may execute within App scope until revoked.  
**Risk status:** CRITICAL.

### T018A — Audit modification/deletion
**Assets:** A005.  
**Attack path:** compromised broker modifies past events.  
**Controls:** broker append-only; no UPDATE/DELETE; external anchor.  
**Risk status:** CRITICAL.

### T018B — Audit omission
**Assets:** A005.  
**Attack path:** compromised broker executes action without writing event.  
**Controls:** fail write if audit unavailable; correlate GitHub events with audit; independent anchor/checkpoints.  
**Risk status:** HIGH.

### T018C — Audit reordering/truncation/clock manipulation
**Assets:** A005.  
**Controls:** sequence IDs, hash chain/checkpoints, trusted time assumptions, reconciliation.  
**Risk status:** MEDIUM/HIGH pending design.

### T018D — Retention/privacy misconfiguration
**Assets:** A005.  
**Attack path:** retain too much/too little, expose sensitive data, lose forensic evidence.  
**Controls:** explicit data fields, retention owner, access policy, deletion/anonymization rules.  
**Risk status:** MEDIUM pending data design.

### T019 — Broker/policy/audit unavailability
**Assets:** A009.  
**Attack path:** service outage/DoS/dependency outage.  
**Impact:** authorized writes unavailable.  
**Candidate controls:** strict fail-closed baseline; operational recovery plan.  
**Enforcement:** broker.  
**Test/evidence:** unavailable policy/audit → write BLOCK.  
**Residual risk:** operational downtime.  
**Risk status:** MEDIUM security / HIGH availability depending use case.

### T020 — Emergency bypass becomes permanent bypass
**Assets:** A001, A003.  
**Attack path:** break-glass credential exists permanently or is over-privileged.  
**Candidate controls:** no break-glass by default; if selected, APPROVED_EXCEPTION, explicit activation, time-bound, minimum permissions, separate audit, auto-expiry, post-incident review.  
**Residual risk:** emergency path itself becomes attack target.  
**Risk status:** HIGH if enabled.

### T021 — Security drift
**Assets:** all.  
**Attack path:** new PAT/App/deploy key/workflow/permission appears after review.  
**Impact:** documented model diverges from reality.  
**Candidate controls:** periodic bottom-up validation; event-triggered revalidation after sensitive changes.  
**Detection:** REALITY vs SPECIFICATION comparison.  
**Test/evidence:** induced drift → SECURITY VIOLATION.  
**Residual risk:** detection interval.  
**Risk status:** HIGH.

### T022 — Repository identity confusion
**Assets:** A001, A002.  
**Attack path:** rename/transfer/similar-name repo causes policy to apply to wrong target.  
**Candidate controls:** immutable `repository_id`; expected owner/name consistency checks.  
**Enforcement:** broker canonicalization.  
**Test/evidence:** mismatch → BLOCK.  
**Residual risk:** GitHub identity assumptions.  
**Risk status:** HIGH.

### T023 — Human admin/owner bypass
**Assets:** A001, A002, A003.  
**Threat actor:** TA008.  
**Attack path:** browser/git/PAT/SSH/admin UI directly changes repo/App settings outside broker.  
**Impact:** bypass main invariant if treated as hidden path.  
**Candidate controls:** explicitly classify human admin paths as APPROVED_EXCEPTION or remove/restrict; audit/review.  
**Residual risk:** ultimate account owner retains platform-level authority unless organizational controls remove it.  
**Risk status:** CRITICAL for AI-invariant interpretation; accepted/controlled for owner governance.

### T024 — Bootstrap/root-of-trust compromise
**Assets:** A002, A003, A006.  
**Threat actor:** TA003, TA004, TA008.  
**Attack path:** initial App creation, policy creation, broker deployment or permission bootstrap is malicious/incorrect before controls exist.  
**Candidate controls:** explicit BOOTSTRAP TRUST procedure, manual review, one-time setup evidence, post-bootstrap lockdown and revalidation.  
**Residual risk:** initial trust cannot be derived from broker before broker exists.  
**Risk status:** CRITICAL during bootstrap.

### T025 — Forensic evidence loss
**Assets:** A005.  
**Attack path:** logs rotate, deployment history missing, credential issuance history unavailable, timestamps inconsistent.  
**Candidate controls:** forensic readiness in incident response, retention policy, configuration snapshots, deployment and credential history.  
**Residual risk:** external systems may have different retention.  
**Risk status:** MEDIUM/HIGH.

### T026 — Canonicalization / interpretation mismatch
**Assets:** A001, A002, A006.  
**Threat actor:** TA002, TA009 nebo neúmyslná klientská nejednoznačnost.  
**Preconditions:** stejný request lze reprezentovat více způsoby nebo parser/policy/executor interpretují data odlišně.  
**Attack path:** representation A → policy canonicalizuje jako stav X → executor interpretuje jinak → vznikne stav Y.  
**Příklady:** path normalization, Unicode/case rozdíly, ref normalization, encoded characters, duplicate/ambiguous fields, repository identity representation, schema/executor disagreement.  
**Impact:** policy autorizuje jinou operaci, než executor skutečně provede.  
**Candidate controls:** C019 single canonical representation; strict schemas; `additionalProperties=false`; normalization před policy evaluation; execution pouze z canonical object.  
**Enforcement:** request canonicalizer + schema validator + executor contract.  
**Detection:** canonical-digest mismatch, parser/schema rejection, security violation.  
**Test/evidence:** ekvivalentní reprezentace musí mít shodný canonical digest; nejednoznačná reprezentace → BLOCK.  
**Residual risk:** chyba ve společném canonicalizeru může ovlivnit policy i executor.  
**Risk status:** CRITICAL.

### T027 — Duplicate / replayed Action Request
**Assets:** A001, A006, A009.  
**Threat actor:** TA009, síťové chyby nebo neúmyslné klientské retry.  
**Preconditions:** klient po timeoutu neví, zda operace proběhla, nebo znovu odešle stejný request.  
**Attack path:** ALLOW request → GitHub execution → timeout/response loss → retry → druhá execution.  
**Impact:** duplicitní branch/PR/commit/merge nebo jiný nekontrolovaný opakovaný side effect.  
**Candidate controls:** C020 request idempotency; unique `request_id`; canonical request digest; execution-state registry; atomic duplicate detection.  
**Enforcement:** broker execution layer.  
**Detection:** duplicate request event + audit correlation.  
**Test/evidence:** stejný `request_id` a canonical request nesmí vytvořit druhý side effect.  
**Residual risk:** externí API může mít vlastní neatomické semantics; broker musí post-checkem zjistit skutečný stav.  
**Risk status:** HIGH.

### T028 — Secret disclosure into untrusted context
**Assets:** A003, A004, A005, A007.  
**Threat actor:** TA002, TA004, TA005, TA006 nebo neúmyslný debug/diagnostic flow.  
**Preconditions:** secret se dostane do exception traceback, debug logu, audit payloadu, ChatGPT contextu, PR/Issue, CI outputu nebo diagnostického bundle.  
**Attack path:** secret-bearing runtime data → untrusted/logging context → další actor secret přečte → credential abuse.  
**Impact:** kompromitace installation tokenu, confirmation/session secretu, externího credentialu nebo signing materialu.  
**Candidate controls:** C021 secret-context isolation and redaction.  
**Enforcement:** logging/audit serializers, exception handling, secret store API, prompt/context construction, CI masking.  
**Detection:** secret-scanning/redaction tests, diagnostic bundle review.  
**Test/evidence:** známý canary secret se nesmí objevit v plaintextu v LLM contextu, auditu, logu ani CI outputu.  
**Residual risk:** secret může uniknout z memory dumpu nebo kompromitovaného hostu mimo běžný logging path.  
**Risk status:** CRITICAL.


---

## 12. GitHub Actions privileged surface

Inventura a v1 musí explicitně posoudit:

- workflow-level `permissions`,
- job-level `permissions`,
- `contents: write`,
- `pull-requests: write`,
- `actions: write`,
- `checks: write`,
- `deployments: write`,
- `packages: write`,
- `security-events: write`,
- `id-token: write`,
- `pull_request_target`,
- `workflow_run`,
- `workflow_dispatch`,
- scheduled workflows,
- reusable workflows,
- third-party Actions,
- secrets,
- PATs/App keys uložené v secrets,
- fork PR behavior,
- runner trust.

Cílové pravidlo:

```text
GitHub Actions
→ žádná privilegovaná capability
  bez explicitně zdokumentované výjimky
```

---

## 13. Confirmation threat surface

`CONFIRMATION-PROTOCOL.md` musí řešit:

- authenticated human channel,
- unique request ID,
- nonce,
- expiry,
- single-use,
- replay detection,
- cancellation,
- concurrency,
- state binding,
- base_ref/base_sha,
- head_ref/head_sha,
- diff/tree digest,
- policy digest,
- request digest,
- merge method,
- session/device-binding decision,
- confirmation UX,
- phishing/approval-fatigue risk.

---

## 14. Credential threat surface

`CREDENTIAL-MODEL.md` musí řešit minimálně:

### GitHub App private key
- storage,
- access,
- exportability,
- signing model,
- rotation,
- revoke,
- compromise procedure.

### Installation token
- lifetime,
- repository scope,
- permission scope,
- issuance,
- redaction,
- no-log policy,
- expiry/revoke.

### Human credentials
- GitHub session,
- PAT,
- SSH keys,
- MFA/passkey decision,
- admin path classification.

### CI credentials
- GITHUB_TOKEN,
- OIDC,
- secrets,
- reusable-workflow credential propagation.

---

## 15. Deployment/host threat surface

`DEPLOYMENT-SECURITY.md` musí rozhodnout:

- kde broker běží,
- pod jakým OS/service principalem,
- kdo má SSH/admin access,
- network exposure,
- firewall,
- patching,
- disk encryption,
- backup protection,
- secret storage,
- signing service,
- deployment identity,
- build identity,
- rollback,
- emergency stop,
- compromise-of-host procedure,
- physical/hosting trust.

---

## 16. Audit, forensics a privacy

Audit musí být minimálně:

```text
append-oriented
tamper-evident
state-bound
externally anchored
```

Musí být určeno:

- kdo zapisuje,
- kdo vlastní,
- kdo čte,
- kdo může mazat,
- kdo spravuje retention,
- kdo spravuje anchor,
- jaká data se ukládají,
- proč se ukládají,
- jaké osobní/citlivé údaje obsahují,
- jak se provádí anonymizace/mazání, pokud je relevantní,
- jak se uchovají forenzní důkazy.

Forenzní připravenost musí zahrnout:
- audit events,
- GitHub events,
- workflow logs,
- broker logs,
- deployment history,
- credential issuance/revocation history,
- configuration snapshots,
- timestamps,
- investigation owner,
- post-mortem process.

---

## 17. Availability a emergency access

Výchozí baseline v0:

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

Před v1 musí být explicitně rozhodnuto:

```text
A) STRICT FAIL-CLOSED
nebo
B) CONTROLLED EMERGENCY ACCESS
```

Pokud bude zvoleno B, vznikne samostatný `BREAK-GLASS.md` a emergency write path bude vedena jako `APPROVED_EXCEPTION`.

---

## 18. Bootstrap / root of trust

Před zavedením brokeru existují operace, které broker nemůže sám autorizovat, protože ještě neexistuje.

Bootstrap zahrnuje minimálně:

- vytvoření GitHub App,
- nastavení App permissions,
- instalaci App,
- vytvoření policy repository/data,
- vytvoření broker deploymentu,
- nastavení secret/signing service,
- vytvoření audit/anchor store,
- přiřazení security ownera.

Tyto operace musí být popsány jako `BOOTSTRAP TRUST`, ručně reviewované a po dokončení musí následovat post-bootstrap validation a lockdown.

---

## 19. Security drift

Bezpečnostní model musí fungovat oběma směry:

```text
SPECIFICATION
↓
IMPLEMENTATION
```

a zároveň:

```text
PRODUCTION REALITY
↓
EFFECTIVE PERMISSIONS
↓
WRITE PATHS
↓
CONTROLS
↓
COMPARE WITH SPECIFICATION
```

Výsledek:

```text
REALITY == SPECIFICATION
→ PASS

REALITY != SPECIFICATION
→ SECURITY VIOLATION
→ INCIDENT RESPONSE
```

Kontrolovat zejména:

- nový PAT,
- novou GitHub App,
- změnu App permissions,
- nový deploy key,
- workflow permission změny,
- nové write paths,
- nové secrets/OIDC capability,
- principals mimo inventory,
- změny deploymentu,
- změny audit ownership/anchor.

---

## 20. Risk register v0

| Threat | Severity | Likelihood | Stav |
|---|---|---|---|
| T001 Prompt injection | HIGH | UNKNOWN | candidate controls defined |
| T002 Direct bypass | CRITICAL | UNKNOWN | inventory required |
| T003 Policy evaluator bug | CRITICAL | POSSIBLE | design controls required |
| T004 Policy compromise | CRITICAL | UNKNOWN | governance required |
| T005 App private key compromise | CRITICAL | UNKNOWN | credential model required |
| T007 Confirmation forgery | CRITICAL | POSSIBLE | protocol required |
| T009 TOCTOU | CRITICAL | POSSIBLE | protocol required |
| T011 Actions privileged path | CRITICAL | UNKNOWN | inventory required |
| T014 Supply chain | HIGH | POSSIBLE | controls required |
| T017 Broker host compromise | CRITICAL | UNKNOWN | deployment design required |
| T018 Audit compromise | CRITICAL | UNKNOWN | audit design required |
| T021 Security drift | HIGH | LIKELY over time | observability required |
| T023 Human admin bypass | CRITICAL | KNOWN POSSIBLE | exception/governance required |
| T024 Bootstrap compromise | CRITICAL | POSSIBLE | bootstrap procedure required |

Likelihood values are provisional until v1.

---

## 21. Candidate control mapping

Definitivní controls budou v `CONTROLS.yaml`. v0 zavádí minimální candidate IDs:

- **C001** No ChatGPT write credential
- **C002** Broker authoritative-state lookup
- **C003** Default deny / unknown deny
- **C004** State-bound human confirmation
- **C005** Replay-safe confirmation
- **C006** GitHub App least privilege
- **C007** Private-key isolation/signing service
- **C008** Write-path inventory and classification
- **C009** Effective-permission drift detection
- **C010** Security-sensitive operation classes
- **C011** Append-only externally anchored audit
- **C012** Actions privilege default deny
- **C013** Supply-chain integrity controls
- **C014** Deployment hardening
- **C015** Emergency App suspend
- **C016** Bootstrap review and lockdown
- **C017** Human admin path governance
- **C018** Untrusted-content / instruction hierarchy
- **C019** Single canonical representation / strict parsing
- **C020** Action-request idempotency and duplicate detection
- **C021** Secret-context isolation and redaction

Každý candidate control musí být ve specifikační fázi převeden na formální control s:
- mitigates,
- enforced_by,
- verification,
- failure_mode,
- residual_risk.

---

## 22. Security violation classes v0

Minimální očekávané classes:

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
V010 Unauthorized privileged Actions capability
V011 Repository identity mismatch
V012 Replay attempt
V013 Bootstrap integrity failure
V014 Canonicalization / interpretation mismatch
V015 Duplicate action request
V016 Secret disclosure to untrusted context
```

Detailní detection/response/owner budou v `SECURITY-VIOLATIONS.yaml`.

---

## 23. Open decisions před v1

Musí být rozhodnuto nebo inventurou potvrzeno:

1. Které GitHub write paths skutečně existují?
2. Které human-admin paths zůstanou jako `APPROVED_EXCEPTION`?
3. Jaké GitHub Apps/PATs/deploy keys/OAuth Apps existují?
4. Jaké effective permissions mají Actions?
5. Kde broker poběží?
6. Jak bude uložen/používán GitHub App private key?
7. Jak bude řešen confirmation authentication?
8. Je nutný session/device binding?
9. STRICT FAIL-CLOSED nebo CONTROLLED EMERGENCY ACCESS?
10. Jaký bude audit store a external anchor?
11. Kdo je security owner, incident owner a audit owner?
12. Jaké retention/privacy požadavky jsou relevantní?
13. Jak bude provedeno bootstrap lockdown?
14. Jaký bude interval drift validation?
15. Jak bude definována canonical representation a parser contract?
16. Jaká budou idempotency semantics Action Requestu?
17. Jak bude enforceováno, že secrets nevstoupí do LLM/log/audit plaintext contextu?

---

## 24. Povinné vstupy pro THREAT MODEL v1

v1 nesmí vzniknout bez:

- `WRITE-PATHS.md`,
- credentials inventory bez secretů,
- GitHub Apps inventory,
- PAT/deploy-key/OAuth inventory,
- GitHub Actions principal inventory,
- workflow/job effective permissions,
- actual repository IDs,
- human-admin path classification,
- deployment target decision,
- confirmation authentication decision,
- audit ownership/design decision,
- availability/emergency-access decision,
- bootstrap state,
- canonicalization/parser contract,
- Action Request idempotency model,
- secret redaction/context-isolation model.

---

## 25. Exit criteria pro THREAT MODEL v0

v0 je připraven pro další krok pouze pokud:

- scope a non-goals jsou definovány,
- security invariants jsou definovány,
- assets jsou definovány,
- actors a threat actors jsou odděleny,
- principals/credentials/effective permissions jsou konceptuálně odděleny,
- trust assumptions jsou definovány,
- TCB je definována,
- trust boundaries mají bezpečnostní význam,
- threat catalogue obsahuje attack path, impact, candidate controls, enforcement, detection, test/evidence a residual risk,
- GitHub Actions privileged surface je zahrnuta,
- confirmation threats jsou zahrnuty,
- credential threats jsou zahrnuty,
- supply-chain threats jsou zahrnuty,
- deployment/host threats jsou zahrnuty,
- audit/forensics/privacy threats jsou zahrnuty,
- availability/emergency threats jsou zahrnuty,
- bootstrap/root-of-trust threats jsou zahrnuty,
- security drift je zahrnut,
- canonicalization/parser-confusion threat je zahrnuta,
- Action Request replay/idempotency threat je zahrnuta,
- secret disclosure do nedůvěryhodného contextu je zahrnuta,
- open decisions jsou explicitní,
- je jasně označeno, co musí potvrdit inventura,
- externí auditní metodiky jsou vedeny jako reference, nikoli controls,
- sandbox assumption pro rizikovou externí validaci je explicitní.

---

## 26. Další krok

Po review tohoto dokumentu:

```text
01-specifikace/02-inventura/WRITE-PATHS.md
+
credentials inventory
+
actual principals/effective permissions
```

Teprve poté vznikne `THREAT MODEL v1`.

---

## 27. Risk evaluation model v0

Risk labels ve v0 jsou **prozatímní** a nesmí být interpretovány jako empirická pravděpodobnost před inventurou.

Každá hrozba má být ve v1 vyhodnocena alespoň podle:

```text
Integrity:       LOW / MEDIUM / HIGH
Confidentiality: LOW / MEDIUM / HIGH
Availability:    LOW / MEDIUM / HIGH
Scope:           single repo / multiple repos / whole system / external system
Reversibility:   reversible / partially reversible / irreversible
Detection time:  immediate / minutes-hours / days / unknown
Likelihood:      likely / possible / unlikely / unknown
```

U každé hodnoty `likelihood` musí být ve v1 uvedeno `likelihood_rationale`. Hodnota `UNKNOWN` je ve v0 preferována všude, kde ji nelze opřít o inventuru nebo ověřenou provozní skutečnost.

Threat ownership pro v0 používá logické role:

- `security_owner` — přijímá/odmítá residual risk a schvaluje bezpečnostní výjimky,
- `architect` — odpovídá za konzistenci modelu,
- `developer` — odpovídá za implementační controls po Definition of Ready,
- `ux_tester` — připravuje nezávislé testy controls/threat scenarios.

Konkrétní fyzické identity budou přiřazeny v `RESPONSIBILITIES.md`.

---

## 28. Assumption validation

| Assumption | Validace | Failure mode |
|---|---|---|
| AS001 GitHub platform | sledovat relevantní GitHub security advisories a zásadní změny API/permission modelu | zpochybnění platform trust → freeze privileged writes a security review |
| AS002 TLS/transport | používat standardní ověřený TLS stack; žádné vypínání cert validation | transport identity/integrity uncertain → BLOCK |
| AS003 Host OS | patching, host hardening, access review | host compromise → emergency stop + incident response |
| AS004 Secret/signing service | access review, audit issuance, integrity monitoring | signing trust uncertain → BLOCK WRITE + rotate/revoke |
| AS005 Security owner | silná autentizace a explicitní role assignment | owner identity uncertain → no privileged approval |
| AS006 Time source | monitorovat rozumnou časovou synchronizaci | expiry/audit ordering uncertain → confirmation/write BLOCK podle dopadu |
| AS007 Audit anchor | oddělený credential a periodická integrity verification | anchor integrity uncertain → audit violation + BLOCK WRITE |
| AS008 Repository identity | GitHub `repository_id` + owner/name consistency check | mismatch → BLOCK |
| AS009 Audit sandbox | před full auditem ověřit isolation profile: no-network/allowlisted env/read-only target/scratch writes/resource limits | sandbox absent → risky validation se nespouští; finding = needs_validation |

---

## 29. Attack scenarios v0

### S001 — Prompt injection → pokus o direct write bypass
1. Nedůvěryhodný repository/web obsah instruuje ChatGPT, aby ignoroval policy.
2. ChatGPT navrhne škodlivý write.
3. ChatGPT nemá write credential.
4. Broker request canonicalizuje a sám načte GitHub state.
5. Policy vrátí BLOCK.
6. Alternativní non-broker write path nesmí být pro AI dostupná.

**Expected:** žádný GitHub write; audit obsahuje zamítnutý request.  
**Pokryté hrozby:** T001, T002, T026, T028.  
**Invariants:** I001, I003, I004, I005, I006, I008.

### S002 — Schválení PR → změna base → pokus o merge
1. Broker vytvoří confirmation request pro konkrétní base/head/diff/policy state.
2. Human potvrdí.
3. Base SHA se změní.
4. Broker před execution načte aktuální state.
5. Confirmation již neodpovídá current state.

**Expected:** confirmation invalid; nový policy evaluation a případně nové confirmation.  
**Pokryté hrozby:** T009.  
**Invariants:** I006, I007.

### S003 — Workflow získá neočekávané write/OIDC privilege
1. Workflow/job je deklarován jako read-only.
2. Repo konfigurace nebo workflow změna mu přidá `contents: write` nebo `id-token: write`.
3. Bottom-up validation porovná effective permission s capability modelem.

**Expected:** SECURITY VIOLATION; affected write path se nesmí tiše považovat za compliant.  
**Pokryté hrozby:** T011, T013, T021.  
**Invariants:** I009.

### S004 — Timeout po úspěšném GitHub write
1. Broker provede povolený write.
2. GitHub operaci přijme.
3. Response se ztratí / klient dostane timeout.
4. Klient odešle stejný request znovu.

**Expected:** broker rozpozná `request_id`/canonical digest a nevytvoří druhý side effect; post-check zjistí skutečný stav.  
**Pokryté hrozby:** T027.

### S005 — Broker host kompromitován
1. Útočník získá broker host.
2. Pokusí se získat App private key/token a manipulovat audit.
3. Private key nemá být lokálně exportovatelný, pokud bude použit sign-only model.
4. Security owner přes GitHub UI suspenduje App installation.
5. Audit history zůstává mimo možnost zpětného přepsání brokerem.

**Expected:** nové broker writes selžou; incident response zachová evidence.  
**Pokryté hrozby:** T005, T017, T018, T025, T028.  
**Invariants:** I010, I011.

---

## 30. Coverage matrix v0

Tato matice je předběžná; finální strojová vazba je v companion souboru `THREAT-MODEL.yaml`.

| Threat | Invariants | Candidate controls | Test/evidence status |
|---|---|---|---|
| T001 | I001,I004,I005,I006,I008 | C001,C002,C018 | specified, not implemented |
| T002 | I001,I002,I003,I013 | C008,C009,C017 | inventory required |
| T005 | I001,I010 | C006,C007,C015,C021 | design required |
| T007 | I005,I007 | C004,C005 | protocol required |
| T009 | I006,I007 | C004 | protocol required |
| T011 | I009 | C009,C012 | inventory required |
| T017 | I010,I011 | C007,C011,C014,C015 | deployment design required |
| T021 | I009 | C008,C009 | observability required |
| T023 | I003,I013 | C017 | governance required |
| T024 | I001,I013 | C016 | bootstrap procedure required |
| T026 | I006,I008 | C019 | schema/parser design required |
| T027 | — | C020 | execution protocol required |
| T028 | I004,I010 | C007,C021 | redaction/context tests required |

Nepokryté nebo částečně pokryté položky nejsou ve v0 chyba; musí být explicitně převedeny do `CONTROLS.yaml` a `TEST-PLAN.md`.

---

## 31. Threat-model ownership a review

**Document owner (logical role):** `security_owner`  
**Model maintainer:** `architect`  
**Independent reviewer:** `ux_tester` / security reviewer podle aktuálního projektu  
**Review cadence:** povinně:
- před přechodem z v0 do inventury,
- po dokončení inventury při tvorbě v1,
- po security-sensitive změně architektury,
- po závažném incidentu,
- při detekci security drift, která mění assumptions/threats/controls.

Konkrétní fyzické přiřazení rolí bude v `RESPONSIBILITIES.md`.


---

## 32. External references

### REF001 — Cloudflare security-audit-skill

**Type:** external reference / audit methodology  
**Repository:** `cloudflare/security-audit-skill`  
**Role in this project:** metodická reference pro security reconnaissance, coverage-led hunting, independent validation a strukturovaný reporting.

REF001 **není security control** a nesmí být použit jako:
- autorita pro GitHub effective state,
- broker runtime decision point,
- policy evaluator,
- credential authority,
- release-gate authority.

Při každém konkrétním použití REF001 se musí zaznamenat:
- repository URL/name,
- source commit nebo immutable version,
- datum použití,
- audit profile,
- scope,
- zda byl dostupný požadovaný sandbox,
- případná omezení / `needs_validation` findings.

Cloudflare-style reconnaissance je záměrně odděleno od naší autoritativní GitHub inventury:

```text
EXTERNAL RECONNAISSANCE
→ source/local-state methodology
→ coverage plan

NEZNAMENÁ

AUTHORITATIVE GITHUB INVENTORY
→ repository_id / installation_id
→ Apps / PATs / deploy keys / OAuth
→ workflow/job effective permissions
→ human/admin write paths
```

---

## 33. External methodology mapping

Tato tabulka je navigační mapping pro použití REF001. Neznamená, že externí metodika sama dané hrozby mitigovala.

| External methodology class | Naše hrozby / oblasti |
|---|---|
| Prompt injection / untrusted instructions | T001, C018 |
| Excessive agency / alternate write capability | T002, T023 |
| Action-confirmation binding | T007, T008, T009, T027 |
| Tool-schema / dispatcher / representation disagreement | T026 |
| Retry / resume / idempotency | T027 |
| Sensitive context / secret extraction | T028, částečně T005/T006 |
| Privileged CI / workflow capability | T011, T012, T013 |
| Supply chain | T014A–T014D |
| Audit/evidence integrity | T018A–T018D, T025 |

**Referenční integrita:** ID T026–T028 se nepřejmenovávají podle terminologie externí metodiky. V tomto projektu již mají stabilní význam:
- `T026` Canonicalization / interpretation mismatch,
- `T027` Duplicate / replayed Action Request,
- `T028` Secret disclosure into untrusted context.

Externí termíny se proto mapují na naše existující stabilní threat IDs místo vytváření kolidujících identifikátorů.

---

## 34. Pořadí práce po review v0

```text
THREAT-MODEL v0
↓
REF001 Cloudflare security-audit — referenční metodika
↓
Cloudflare-style RECONNAISSANCE
↓
NAŠE AUTORITATIVNÍ GITHUB INVENTURA
↓
coverage ledger inventury
↓
THREAT-MODEL v1
↓
CONTROLS / POLICY / PROTOCOLS
↓
implementace step-by-step
↓
developer tests
↓
independent/full security audit
↓
UX / acceptance tests
↓
RELEASE GATE
```

External audit ani reconnaissance nenahrazuje žádnou autoritativní inventuru, control ani release decision.
