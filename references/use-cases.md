# Verified PSL Use Cases — with primary-source citations

Every entry below was verified against the named primary source (2026-09-01): RFCs, WHATWG/W3C specs, Chromium/Gecko/Boulder source at `main`, first-party vendor docs, live TLS handshakes. Chromium citations name the function; line numbers drift, symbols don't.

## Browser security boundaries ("site" = scheme + eTLD+1)

1. **Cookie `Domain=` public-suffix rejection** — RFC 6265 §5.3 step 5 ("essential for preventing…"); rfc6265bis §5.7 step 9 ("if the user agent is configured to reject public suffixes"). Chromium `GetEffectiveDomain` (net/cookies/cookie_util.cc) uses `INCLUDE_PRIVATE_REGISTRIES`.
   https://www.rfc-editor.org/rfc/rfc6265#section-5.3 · https://httpwg.org/http-extensions/draft-ietf-httpbis-rfc6265bis.html
2. **`document.domain` cannot widen to a public suffix** — HTML "is a registrable domain suffix of or is equal to", step 4. (Legacy: `document.domain` is deprecated, off by default.)
   https://html.spec.whatwg.org/multipage/browsers.html#is-a-registrable-domain-suffix-of-or-is-equal-to
3. **SameSite enforcement** — rfc6265bis §5.2 → HTML "obtain a site" → URL spec registrable domain; Chromium `SchemefulSite` (net/base/schemeful_site.cc, INCLUDE_PRIVATE).
   https://html.spec.whatwg.org/multipage/browsers.html#obtain-a-site · https://url.spec.whatwg.org/#host-registrable-domain
4. **Sec-Fetch-Site** — W3C Fetch Metadata §2.3 (registrable-domain matching spelled out in its §4.1 example).
   https://w3c.github.io/webappsec-fetch-metadata/#sec-fetch-site-header
5. **Site Isolation (Chromium)** — site = scheme + registered domain; `SiteInfo::GetSiteForOrigin` (content/browser/site_info.cc, INCLUDE_PRIVATE).
   https://www.chromium.org/Home/chromium-security/site-isolation/
6. **Storage partitioning (Chrome)** — partition key (top-level site, origin).
   https://privacysandbox.google.com/cookies/storage-partitioning
7. **HTTP cache / network-state partitioning (Chrome)** — key includes top-level site scheme://eTLD+1.
   https://developer.chrome.com/blog/http-cache-partitioning
8. **State partitioning (Firefox)** — double-keyed by top-level site; Gecko builds its eTLD table from the whole PSL, no ICANN/PRIVATE distinction (netwerk/dns/prepare_tlds.py; caps/OriginAttributes.cpp `SetPartitionKey`).
   https://developer.mozilla.org/en-US/docs/Web/Privacy/Guides/State_Partitioning
9. **WebAuthn / passkey RP ID validation** — a tenant page cannot widen its RP ID up to a shared public suffix (WebAuthn L3 "Relying Party Identifier" + HTML algorithm step 4).
   https://www.w3.org/TR/webauthn-3/#relying-party-identifier

## Certificates

10. **Chromium wildcard-cert check** — `X509Certificate::VerifyHostname` (net/cert/x509_certificate.cc): rejects `*.com`/`*.co.uk`; comment permits `*.appspot.com` (`EXCLUDE_PRIVATE_REGISTRIES`).
11. **Let's Encrypt issuance policy** — ICANN section only: `iana.ExtractSuffix` (`IgnorePrivate: true`, iana/iana.go); `policy/pa.go` ("No `*.com`!"). Precedent: wildcard rule `*.user.fm` listed 2022 (publicsuffix/list PR #1601), LE `*.user.fm` certs issued through 2026 (crt.sh); live `*.github.io` is LE-issued while `github.io` is PRIVATE-listed.
    https://github.com/letsencrypt/boulder/blob/main/iana/iana.go · https://github.com/letsencrypt/boulder/blob/main/policy/pa.go · https://crt.sh/?CN=%2A.user.fm
12. **Let's Encrypt rate limits** — per registered domain; Boulder `coveringIdentifier` (ratelimits/utilities.go) uses the full PSL (publicsuffix-go default `IgnorePrivate: false`) — PRIVATE entries get per-subdomain buckets.
    https://letsencrypt.org/docs/rate-limits/ · https://github.com/letsencrypt/boulder/blob/main/ratelimits/utilities.go

## Credential & settings UI

13. **Password-manager autofill scope (Chrome)** — `IsPublicSuffixDomainMatch` (components/password_manager/core/browser/password_store/psl_matching_helper.cc, INCLUDE_PRIVATE).
14. **Site-settings & delete-browsing-data grouping (Chrome)** — origins grouped by eTLD+1 (`GroupingKey`/`GetEtldPlusOneForHost`, chrome/browser/ui/webui/settings/site_settings_handler.cc). Permission *grants* remain origin-scoped.
15. **Related Website Sets** — members were required to be eTLD+1 per a PSL snapshot (RWS Submission Guidelines). RWS was retired by Google in Oct 2025 with most of the Privacy Sandbox (privacysandbox.google.com/blog/update-on-plans-for-privacy-sandbox-technologies); listed as a historical consumer.
    https://github.com/GoogleChrome/related-website-sets/blob/main/RWS-Submission_Guidelines.md

## URL display

16. **"www" hiding (Chrome)** — `StripWWW` (components/url_formatter/url_formatter.cc) calls `GetDomainAndRegistry` (INCLUDE_PRIVATE): intranet hosts left alone; `www.` never stripped out of the registrable domain itself.
17. **Hostname elision (Chrome)** — `SplitHost` (components/url_formatter/elide_url.cc) splits subdomains from domain-and-registry so shortening drops leading labels and always preserves the eTLD+1. (Note: Chrome's omnibox emphasis is the whole hostname — the PSL is NOT used for eTLD+1 highlighting in Chrome.)

## Ecosystem tooling

18. **curl** — built with libpsl by default; without it curl "has no ability to stop super cookies".
    https://github.com/curl/curl/blob/master/docs/HTTP-COOKIES.md
19. **wget** — "Use libpsl for verifying cookie domains" (NEWS, 1.16).
20. **PSL libraries** — tldextract (Python), golang.org/x/net/publicsuffix (Go), libpsl (C; README lists canonical use cases).
21. **Google Search Console** — Domain properties bounded by the public suffix; "We don't support domain-property verification of public suffixes" — the words "public suffixes" hyperlink to publicsuffix.org (visible only in the HTML).
    https://support.google.com/webmasters/answer/34592
22. **GA4 cookie-domain detection** — `cookie_domain: 'auto'` = "top level domain plus one subdomain (eTLD +1)"; Google's server-side Tag Manager API doc names the PSL explicitly.
    https://developers.google.com/analytics/devguides/collection/ga4/reference/config#cookie_domain · https://developers.google.com/tag-platform/tag-manager/server-side/api
23. **DMARC (deployed)** — RFC 7489 §3.2 organizational domain via the PSL; obsoleted by RFC 9989 (May 2026, DNS Tree Walk) — cite in deployed-standard tense.
    https://www.rfc-editor.org/rfc/rfc7489.html#section-3.2 · https://www.rfc-editor.org/rfc/rfc9989.txt
24. **Safe Browsing v5 host expressions** — spec: "The determination of eTLD+1 should be based on the Public Suffix List"; clients hash the eTLD+1 plus up to three longer suffixes and the exact host. Chromium `GenerateHostVariantsToCheckV5` (components/safe_browsing/core/browser/db/sb_protocol_manager_util.cc) calls `GetDomainAndRegistry(host, EXCLUDE_PRIVATE_REGISTRIES)` — ICANN section only, so a PRIVATE entry does not change which suffixes Chrome checks. v4 (`GenerateHostVariantsToCheckV4`) used the last five labels with no PSL lookup.
    https://developers.google.com/safe-browsing/reference/URLs.and.Hashing · https://chromium.googlesource.com/chromium/src/+/refs/heads/main/components/safe_browsing/core/browser/db/sb_protocol_manager_util.cc

## Claims that did NOT survive verification (do not assert)

- Chrome omnibox eTLD+1 highlighting (emphasis code has no PSL call).
- Spamhaus DBL / URIBL PSL usage (Spamhaus FAQ says "lists at the main domain level" — quotable, but no PSL mention).
- Firefox Multi-Account Containers (per-hostname assignment, no PSL).
- Permission grants keyed by eTLD+1 (origin-scoped; only the settings UI groups by eTLD+1).
- Search ranking / spam propagation respecting PSL boundaries (no first-party source).
- Third-party password managers' PSL usage (unverified — say "Chrome's password manager").
