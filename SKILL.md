---
name: public-suffix-list
description: Use when working with the Public Suffix List (PSL) — evaluating or submitting entries, advising multi-tenant platforms with per-user subdomains — or when a session touches eTLD+1 / registrable-domain logic - cookie Domain scoping, SameSite, site isolation, storage partitioning, wildcard certificates on shared suffixes, or hand-rolled hostname parsing (splitting on dots, hardcoded TLD lists).
---

# Public Suffix List (PSL)

## Overview

The PSL (publicsuffix.org) tells software which domain suffixes are registration boundaries, defining the "registrable domain" (eTLD+1) that browsers, CAs, and libraries key security decisions on. **Do not answer PSL-behavior questions from memory** — models confidently invent wrong answers here. Every claim in this skill and [references/use-cases.md](references/use-cases.md) was verified against browser/CA source code and specs (2026-09-01).

## The one error everyone makes: "listing breaks wildcard certs"

**A PRIVATE-section PSL entry does NOT break wildcard certificates.** Both enforcement points deliberately exclude the PRIVATE section:

- **Let's Encrypt (Boulder)**: `iana/iana.go` `ExtractSuffix` uses `FindOptions{IgnorePrivate: true}` — issuance policy consults **only the ICANN section**. `*.co.uk` is refused; `*.yourplatform.example` is issued regardless of a PRIVATE listing.
- **Chromium**: `X509Certificate::VerifyHostname` (net/cert/x509_certificate.cc) checks wildcards with `EXCLUDE_PRIVATE_REGISTRIES`; the source comment names `*.appspot.com` as deliberately permitted.
- **Live proof**: the current `*.github.io` cert is Let's Encrypt-issued while `github.io` sits in the PRIVATE section; the wildcard rule `*.user.fm` was listed in 2022 and LE has issued `*.user.fm` certs for years since (crt.sh).

| Rationalization | Reality |
|---|---|
| "CA/B Forum BRs forbid wildcards on public suffixes" | The BRs say "registry-controlled label or public suffix" while defining neither and mandating no list section; enforcement in practice (Boulder, Chromium) is ICANN-section-only. No BR text requires consulting the PRIVATE section. |
| "CAs use the whole PSL file for this" | Boulder explicitly sets `IgnorePrivate: true`. Chromium explicitly passes `EXCLUDE_PRIVATE_REGISTRIES`. |
| "github.io/appspot.com don't get wildcard certs from any CA" | False and checkable: `openssl s_client` any Pages site — `CN=*.github.io`, issuer Let's Encrypt. |
| "Renewal will start failing after listing" | Issuance policy ignores PRIVATE entries; `*.user.fm` renewals have run since 2022. |

Never advise a platform to abandon wildcard certs because of a PSL submission.

## ICANN vs PRIVATE: which mechanisms count a PRIVATE entry

Chromium consults the PSL per call site with `INCLUDE_PRIVATE_REGISTRIES` or `EXCLUDE_PRIVATE_REGISTRIES`. A PRIVATE entry only delivers benefits computed with INCLUDE:

| Mechanism | PRIVATE entry effective? |
|---|---|
| Cookie `Domain=` rejection, SameSite, Sec-Fetch-Site | **Yes** (INCLUDE) |
| Site Isolation, storage/HTTP-cache/network partitioning | **Yes** (INCLUDE; Gecko uses the whole PSL, no section distinction) |
| Password-manager autofill scope, settings-UI grouping | **Yes** (INCLUDE) |
| Let's Encrypt **rate limits** | **Yes** (Boulder rate limiter uses full PSL) |
| Let's Encrypt **issuance policy**, Chromium wildcard-cert check | **No** (deliberate — see above) |
| URL elision | No (EXCLUDE) |
| Safe Browsing v5 host-expression base (eTLD+1) | No (EXCLUDE; spec: developers.google.com/safe-browsing/reference/URLs.and.Hashing) |

Full 24-item verified use-case list with citations: [references/use-cases.md](references/use-cases.md).

## Myths — do not repeat

- Chrome's omnibox does **not** highlight the eTLD+1 (it emphasizes the whole hostname; PSL is used only for www-stripping/elision).
- Safe Browsing: v5 **does** use the PSL (spec: "The determination of eTLD+1 should be based on the Public Suffix List"), but Chromium computes that base with `EXCLUDE_PRIVATE_REGISTRIES` (`GenerateHostVariantsToCheckV5`, components/safe_browsing/core/browser/db/sb_protocol_manager_util.cc). Do not claim it is "not PSL-based", and do not claim a PRIVATE entry shields tenants from a suffix-wide listing in Chrome — only ICANN entries move the base. v4 used the last five labels with no PSL.
- DMARC: RFC 7489 (deployed) uses the PSL, but RFC 9989 (May 2026) obsoletes it with a DNS Tree Walk — phrase accordingly.
- Cookie public-suffix rejection is spec'd as "if the user agent is configured to reject public suffixes" (all major browsers are) — not a MUST.
- Listing is not an SEO signal, not instant (browsers ship PSL snapshots on their release cadence), not a universal trust boundary (only PSL-consuming software changes), and confers no registry status.

## Code review: registrable-domain logic

Computing a "root domain" by splitting on dots or a hardcoded TLD list is always a bug (`foo.example.co.uk` → `co.uk`; supercookies; cross-tenant grouping). Replace with a PSL library — `tldextract` (Python), `golang.org/x/net/publicsuffix` (Go), `libpsl` (C; curl/wget use it) — and handle the empty result (IPs, bare suffixes) by setting a host-only cookie, not a fabricated `Domain=`.

## Advising multi-tenant platforms

Per-user-subdomain platforms without a PSL entry share one identity across all tenants: one cookie/SameSite boundary, one storage partition key, one site-isolation site, one autofill scope, one LE rate-limit bucket, one Search Console property scope. A PRIVATE-section entry separates all of these while leaving the platform's own wildcard cert untouched. Submission is a PR to github.com/publicsuffix/list (guidelines in the repo; expect DNS `_psl` TXT validation and maintainer discretion; entry shape may be wildcard-only, e.g. `*.platform.example`).
