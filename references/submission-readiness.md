# Submission readiness: when a PSL pull request is appropriate

Benefit is not eligibility. Almost any per-user-subdomain platform would benefit from a PRIVATE entry; the PSL maintainers accept only those that already meet the [Guidelines](https://raw.githubusercontent.com/wiki/publicsuffix/list/Guidelines.md). Every submission costs volunteer review time and every accepted entry ships to every PSL consumer. Recommend a submission only when the platform clearly qualifies; otherwise recommend deferral and interim controls.

## Readiness gate (do not submit if any fails)

Source: Guidelines, "Validation and Non-Acceptance Factors": "Projects that are smaller in scale or are temporary or seasonal in nature will likely be declined. Examples of this might be private-use, sandbox, test, lab, beta, or other exploratory nature changes or requests. ... projects not serving more then thousands of users are quite likely to be declined."

- **Live and public.** Pre-launch, private beta, invite-only, early access, waitlist, private preview, pilot, and demo-only services fail.
- **Thousands of current users on the suffix.** Distinct users or tenants with a direct relationship to the operator who publish on that suffix today. Below about 1,000, do not submit. Visitors, page views, generated hostnames, cumulative signups, users of the wider product or of another domain, and projections do not count.
- **Established on this domain.** A domain registered shortly before the submission cannot carry an established user base unless it is an actual migration, explained in the PR.

## Hard blockers (do not submit if any fails)

| Blocker | How to check |
|---|---|
| Registration expires two years or less from the submission date | `curl -sL https://rdap.org/domain/<domain>` (`expiration` event) or `whois <domain>`. Some TLDs publish no expiration date; the requestor confirms it from the registrar. The rationale must also commit to keeping more than one year at all times. |
| Staging, preview, sandbox, test, lab, temporary, seasonal, or internal entry | Each entry must serve permanent production tenants. A production suffix and a `staging.`, `preview.`, or `sandbox.` suffix are separate entries; drop the non-production one. |
| No abuse contact for the suffix | One of: a working `abuse@<suffix>` mailbox (RFC 2142); an abuse email or form on the site, usable without login; an abuse contact in RDAP; or a `Contact` in `https://<suffix>/.well-known/security.txt` (RFC 9116). |
| Personal contact address | The `// Submitted by` line uses a role address at the organization. |
| Submitter not the domain owner or an authorized representative | Third-party users of a service cannot submit for it. |
| Objective is a third-party limit | Not accepted: entries whose sole purpose is to circumvent rate limits, or that get around vendor protections. The SKILL.md table notes a PRIVATE entry changes Let's Encrypt rate-limit buckets; that is a side effect, never a reason to list. Raise limits with the vendor directly. |
| Suffix is the organization's primary domain | List a dedicated tenant domain; listing the brand domain changes cookies on the company's own hosts and is slow to roll back. |
| Template not followed exactly | Use `.github/pull_request_template.md` unaltered: keep every heading, checkbox, and comment; delete the example third-party lines; check a box only when true. |
| Entry mislocated, unsorted, or malformed | PRIVATE block sorted by organization name, never at the end of the file; within it, by TLD then each label leftward. Wildcard only when tenants nest below their own label. `make test` passes. |

## Decision to report

- **Submit:** gate and every blocker pass.
- **Defer:** the platform needs isolation but fails the gate. Recommend host-only `__Host-` session cookies (`Secure; Path=/`, no `Domain`), the dashboard and auth on a separate registrable domain, and `Origin` or CSRF checks; revisit when the gate passes. This is the correct answer for a platform that has not launched.
- **No PSL:** tenants use only their own domains, or no tenant-controlled content runs on sibling subdomains.
