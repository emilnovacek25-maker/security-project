# Security Project — GitHub Policy Broker

Stav: **SPECIFIKAČNÍ FÁZE / IMPLEMENTACE BROKERU ZATÍM NEZAHAJOVAT**

Tento repozitář je veden jako projekt podle vývojových fází. Kořen obsahuje pouze projektový přehled; pracovní artefakty patří do složek podle fáze projektu.

## Struktura projektu

```text
security-project/
├── README.md
├── 00-zadani-a-oponentura/
├── 01-specifikace/
│   ├── 01-threat-model/
│   ├── 02-inventura/
│   ├── 03-actor-capability-model/
│   ├── 04-controls-a-violations/
│   ├── 05-credentials/
│   ├── 06-protokoly-a-schemata/
│   ├── 07-deployment-a-audit/
│   ├── 08-incident-a-odpovednosti/
│   └── 09-migration-test-observability/
├── 02-implementace/
├── 03-nezavisle-testovani/
├── 04-nasazeni/
├── 05-provoz-a-drift/
└── 90-archiv/
```

Git neukládá prázdné složky, takže další adresáře budou vznikat postupně s prvními artefakty dané fáze.

## Nejvyšší bezpečnostní invariant

> Žádná GitHub write operace, která podle bezpečnostního modelu podléhá brokeru, nesmí být technicky proveditelná jinou cestou než prostřednictvím brokeru.

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

## Aktuální fáze

```text
KONCEPČNÍ OPONENTURA
→ UZAVŘENA

NORMATIVNÍ ZADÁNÍ
→ READY

SPECIFIKAČNÍ FÁZE
→ PROBÍHÁ

FORMÁLNÍ BEZPEČNOSTNÍ SPECIFIKACE
→ NOT COMPLETE

IMPLEMENTACE BROKERU
→ ZATÍM NEZAHAJOVAT
```

Aktuální první artefakt:
`01-specifikace/01-threat-model/THREAT-MODEL.md`

Další krok:
`01-specifikace/02-inventura/WRITE-PATHS.md` + credentials inventory.
