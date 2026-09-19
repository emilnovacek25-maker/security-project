# AUTHORITATIVE-INVENTORY-CHECKLIST.md

Status: CHECKLIST / INVENTORY NOT YET COMPLETE  
Phase: 01-specifikace/02-inventura

## 1. Účel

Toto je autoritativní inventura skutečného GitHub effective state.

Na rozdíl od source reconnaissance musí být každý bezpečnostně relevantní údaj podložen live GitHub stavem nebo jiným autoritativním zdrojem.

Základní řetězec:

```text
LOGICAL ACTOR
↓
AUTHENTICATION PRINCIPAL
↓
CREDENTIAL
↓
EFFECTIVE PERMISSION
↓
WRITE PATH
↓
CLASSIFICATION
```

Každá write cesta musí skončit jako:

```text
BROKERED
APPROVED_EXCEPTION
REMOVED
```

`UNKNOWN` je dočasný stav inventury, nikoli přijatelný cílový stav.

## 2. Repository inventory

Pro každý repository target zjistit:

- repository_id,
- owner_id, pokud dostupné/relevantní,
- owner/name,
- visibility,
- default branch,
- archived/disabled state,
- rulesets/branch protections,
- kdo může admin/write/maintain,
- zda je repo součástí scope brokeru.

Evidence:
- source,
- timestamp,
- actor, který údaj načetl.

## 3. Human principals

Inventarizovat:

- owners/admins,
- collaborators,
- org/team membership, pokud relevantní,
- role/effective permissions,
- browser/UI write capability,
- direct git push capability,
- merge capability,
- settings/App-management capability.

Neinventarizovat hesla ani session secrets.

## 4. GitHub Apps

Pro každou App:

- App name/id,
- installation_id,
- owner/account/org,
- repository access scope,
- repository permissions,
- organization permissions,
- token issuance model,
- kdo může App suspend/uninstall/configure,
- zda může zapisovat,
- write paths, které umožňuje.

## 5. PATs

Zjistit existenci relevantních:

- fine-grained PAT,
- classic PAT,
- owner,
- repository scope,
- permissions/scopes,
- expiry,
- purpose,
- storage location category,
- whether still required.

Nikdy neukládat token value.

## 6. SSH / deploy keys

Inventarizovat:

- user SSH keys relevantní pro write,
- deploy keys,
- read-only vs write-enabled,
- repository binding,
- owner/purpose,
- poslední známé použití, pokud dostupné.

## 7. OAuth Apps / external services

Inventarizovat:

- OAuth Apps,
- third-party GitHub Apps,
- bots,
- external automation,
- CI/CD systems,
- webhooks, pokud vedou k privileged action,
- credential type,
- effective GitHub capability.

## 8. GitHub Actions principals

Pro každý workflow a relevantní job:

- path,
- trigger,
- workflow-level `permissions`,
- job-level `permissions`,
- effective `GITHUB_TOKEN` capability,
- `id-token: write`,
- secrets used,
- PAT/App keys referenced,
- reusable workflows,
- third-party Actions,
- mutable vs immutable refs,
- fork PR behavior,
- `pull_request_target`,
- `workflow_run`,
- `workflow_dispatch`,
- schedules,
- runner type/trust.

Každý write-capable workflow/job je samostatný principal record.

## 9. Branch/ruleset enforcement

Pro každý chráněný target:

- required PR?,
- required reviews?,
- status checks?,
- force-push allowed?,
- deletion allowed?,
- bypass actors?,
- ruleset scope?,
- merge restrictions?,
- direct push actors?,
- admin bypass behavior?

Ruleset je defense in depth; nesmí být zaměněn za broker policy.

## 10. Write-path enumeration

Hledat minimálně:

- REST API contents write,
- Git Database API: blob/tree/commit/ref,
- GraphQL mutations,
- git over SSH,
- git over HTTPS,
- browser/UI edits,
- merge PR,
- squash/rebase merge,
- branch creation/deletion,
- tag/ref update,
- release/package/deployment writes pokud relevantní,
- Actions-driven writes,
- App/bot writes,
- external automation writes,
- admin/settings changes měnící enforcement.

Každý write path record musí uvést:

```text
WRITE_PATH_ID
actor
principal
credential_type
target
operation
effective_permission
current_state
target_classification
evidence
timestamp
threat_ids
notes
```

## 11. Credential inventory

Každý credential record:

```text
CREDENTIAL_ID
type
owner/principal
purpose
scope
effective_permissions
storage_class
exportable?
expiry
rotation_owner
revoke_method
related_write_paths
status
evidence
```

Bez secret value.

## 12. Effective permission verification

Nestačí deklarované permissions.

Pro každý principal ověřit:

```text
DECLARED CAPABILITY
vs
EFFECTIVE PERMISSION
```

Mismatch:

```text
→ V001 Declared capability mismatch
→ SECURITY VIOLATION
```

## 13. Human/root governance

Explicitně klasifikovat cesty typu:

- account owner,
- organization owner,
- repository admin,
- App installer/configurator,
- branch/ruleset bypass actor.

Žádná z nich nesmí zůstat skrytou implicitní výjimkou.

## 14. Evidence discipline

Každý fakt má mít:

- authoritative source,
- retrieved_at,
- repository/account context,
- raw identifier pokud je bezpečné jej uchovat,
- reviewer status.

Nespoléhat na starý screenshot nebo paměť, pokud lze získat live state.

## 15. Exit criteria

Autoritativní inventura je hotová, když:

- všechny scoped repositories mají immutable identity,
- všechny známé human/machine principals jsou evidovány,
- všechny relevantní credentials jsou evidovány bez secret values,
- všechny Actions principals mají effective permissions,
- všechny write paths jsou evidovány,
- všechny write paths mají cílovou klasifikaci,
- není žádná `UNKNOWN` write path bez otevřeného security violation/incident recordu,
- evidence je dostatečná pro aktualizaci THREAT MODEL v1.
