# public-suffix-list — an agent skill for PSL work

A skill for AI coding agents (Claude Code and other [agentskills.io](https://agentskills.io)-compatible runtimes) that keeps them accurate when working with the [Public Suffix List](https://publicsuffix.org/): evaluating or submitting PSL entries, advising multi-tenant platforms with per-user subdomains, and reviewing code that computes registrable domains (eTLD+1).

## Why this exists

Models answer PSL questions from memory, and memory is confidently wrong here. In baseline testing, an agent asked whether a PRIVATE-section PSL listing would break a platform's Let's Encrypt wildcard certificate answered **"Yes, it will break your wildcard"** — inventing a CA/Browser Forum prohibition, claiming CAs consult the whole PSL, and asserting that `*.github.io` wildcard certs don't exist (the live `*.github.io` cert is Let's Encrypt-issued while `github.io` sits in the PRIVATE section). It then advised migrating to per-tenant certificates — expensive, wrong advice.

With this skill loaded, the same scenario produces the correct answer: Let's Encrypt's issuance policy consults only the ICANN section (Boulder `IgnorePrivate: true`), Chromium's wildcard check passes `EXCLUDE_PRIVATE_REGISTRIES`, and a PRIVATE entry changes browser security boundaries — not certificate issuance.

## What's inside

- **`SKILL.md`** — the operating guidance: the ICANN-vs-PRIVATE section distinction, which browser mechanisms honor a PRIVATE entry (`INCLUDE_PRIVATE_REGISTRIES` vs `EXCLUDE` per Chromium call site), the wildcard-certificate misconception and its counters, myths not to repeat, code-review guidance for registrable-domain logic, and advice for multi-tenant platforms.
- **`references/use-cases.md`** — 24 verified PSL use cases, each cited to a primary source (RFCs, WHATWG/W3C spec anchors, Chromium/Gecko/Boulder source with function names, first-party vendor docs), plus a list of widely repeated claims that did **not** survive verification.

Every factual claim was verified against primary sources on 2026-09-01: specs at their canonical URLs, browser source at `main` (source.chromium.org, searchfox.org), Boulder source, live TLS handshakes, and crt.sh. No secondary sources.

## Install

**Claude Code** — copy this directory into your personal skills directory:

```sh
cp -r psl-skill ~/.claude/skills/public-suffix-list
```

**Codex / Copilot CLI / Gemini CLI** — these also recognize the cross-runtime alias:

```sh
cp -r psl-skill ~/.agents/skills/public-suffix-list
```

The skill triggers on PSL work and on sessions touching eTLD+1 / registrable-domain logic: cookie `Domain` scoping, SameSite, site isolation, storage partitioning, wildcard certificates on shared suffixes, or hand-rolled hostname parsing.

## Caveats

- Citations to browser source name functions rather than line numbers (symbols are stable; lines drift). Source behavior described is as of verification date; re-verify before relying on it adversarially.
- The skill deliberately contains no executable code — it is documentation only.

## License

MIT — see [LICENSE](LICENSE).
