# Hermes Web WL Gate

A practical English guide to **web-based NFT whitelist (WL) gates** — custom project sites with puzzles, social proof, EVM address forms, PoW / score games, and **multi-account** submission from one ops box.

> Built for **Hermes Agent** workflows. This is an ops playbook (skill-level), not a spam kit.  
> Automating engagement can break platform rules. Prefer real accounts, honest proofs, and human pacing.

**Repo:** https://github.com/didin-lab/hermes-web-wl-gate

---

## What this is

Modern WL apps often live on a **project website**, not only on X:

```text
01 unlock gate (puzzle / game / doodle metrics)
02 social proof (follow / RT / quote / reply URL)
03 EVM wallet
04 SUBMIT → API / Supabase / form backend
```

Naive browser automation fails on:

- React state not in the DOM (tile puzzles)
- `localStorage` “already submitted” after first wallet
- Console blocked for form fill / storage reads
- Datacenter TLS → `429` after 3–4 accounts
- X reply filters on `@mention` / write caps (`226`, `344`)

This guide covers how Hermes stacks skills to get through those gates **per account**, with real proof URLs and real API responses.

---

## Features

### 1) Form-type triage (first 30 seconds)
| Signal | Backend | Submit path |
|---|---|---|
| `docs.google.com/forms` | Google Forms | `formResponse` POST |
| `*.supabase.co/rest/v1/...` | Supabase | REST insert + anon JWT |
| `/api/submit`, `/api/wl/apply` | Custom Next.js / SPA | JSON POST (reverse from Network / bundle) |
| Apps Script `script.google.com/macros` | GAS | FormData POST (avoid `no-cors` hang) |
| Only `localStorage` “registered” | Cosmetic client WL | Real X tasks only — no server row |

### 2) Puzzle gates (React fiber + swap plan)
- Tile state is often **not** in visible text → walk `__reactFiber$` for the array
- Compute deterministic swap sequence → click tiles via console
- Form fields still use real `browser_type` / `browser_click` when console is security-blocked

### 3) Default multi-account site transport: **CloakBrowser**
For custom `/api/*` waves (cookie session, PoW, score gates):

- Prefer **CloakBrowser** in-page `fetch` (Win32/Chrome fingerprint)
- Bare Python `requests` from a VPS IP often dies after a few OK submits (`429` / “too many sessions”)
- **X stays on `x.py` / REST follow workaround** — Cloak is for the **site half**

### 4) Social proof without inventing URLs
- Do real follow / RT / like / quote / reply when the backend or manual review checks them
- Per-account **humanized** text (no clone spam)
- If `@mention` reply soft-fails (HTTP 200, empty tweet result), retry **without** mention
- **Never invent** tweet IDs / quote URLs for the form

### 5) API reverse + bypass of local-only locks
After first submit, sites may lock UI via `localStorage`. Multi-wallet ops:

1. Capture real endpoint + body from Network / JS bundle  
2. Submit remaining wallets via API (or Cloak in-page fetch)  
3. Log HTTP status + response proof per account  

### 6) Harder gates (class patterns)
| Gate class | What to reverse | Prefer |
|---|---|---|
| SHA-256 PoW challenge | `nonce` + difficulty → `counter` + `solvedHash` | Solve + ~5s dwell before submit |
| Canvas “draw to unlock” | metrics JSON, not pixels | Mid-strength metrics → `verificationToken` |
| Play-to-score unlock | timed `progress` API | Honor server time gates; skip flaky canvas play |
| Public Supabase / GForm | anon key + field map | Bare REST OK for many cases |

---

## Why this stack

| Approach | Result |
|---|---|
| Manual 8× browser | Slow, easy to miss fields |
| Bare curl only | Misses cookie/TLS gates; 429 waves |
| Blind UI clicker | Fails on React/puzzle/PoW |
| **Triage → reverse API → Cloak for site + x.py for X** | Scale with real proofs |

Benefits:

- Multi-account coverage without one-tab-per-wallet forever  
- Recover after UI “already applied”  
- Separate **site throttle** from **X write budget**  
- Honest logs (`~/wallets/<project>_wl_submissions.json`)

---

## When to use

Use when:

- WL URL is a **custom site** (puzzle, quest steps, wallet field)
- You need **N accounts × wallet map** in one session
- First submits OK, then **429 / session limit**
- Form needs **real X proof URL** + EVM address

Use something else when:

- Pure OpenSea SeaDrop allowlist → [opensea-eligibility-check](https://github.com/didin-lab/opensea-eligibility-check)  
- Pure X multi-account ops → [hermes-x-multi-account](https://github.com/didin-lab/hermes-x-multi-account)  
- Mint after WL → [hermes-nft-skills](https://github.com/didin-lab/hermes-nft-skills)

---

## End-to-end flow

```text
1. Recon page + Network + JS bundle
2. Classify backend (GForm / Supabase / /api/* / cosmetic)
3. Do X tasks per healthy account (x.py) → real proof URLs
4. Solve gate (puzzle / PoW / metrics / score API)
5. Submit per wallet (Cloak default for custom APIs)
6. On 429 → cloak + spacing; never fake proof
7. Save JSON: account, wallet, status, response, quote_url
```

### Minimal payload shapes (examples)

**Custom JSON (conceptual):**
```json
{
  "wallet": "0x...",
  "quoteUrl": "https://x.com/<handle>/status/<id>",
  "referrer": "optional_ref",
  "company": ""
}
```
(`company` / honeypot fields must stay empty when present.)

**Supabase insert (conceptual):**
```http
POST https://<project>.supabase.co/rest/v1/<table>
apikey: <anon jwt>
Authorization: Bearer <anon jwt>
Content-Type: application/json
Prefer: return=representation
```

---

## Multi-account rules of thumb

| Layer | Rule |
|---|---|
| X | Humanized text; space writes; respect `226`/`344` cooldowns |
| Site | Cloak for multi-wave custom APIs; bare REST only if proven public |
| Proof | One real URL per account; fresh URL if server burned the last one |
| Logging | File under `~/wallets/…_wl_submissions.json` — no invented IDs |
| Secrets | Tokens/cookies `chmod 600`; never commit |

Wallet map for X-linked ops is often:

| Account | Wallet label |
|---|---|
| illumi | wcore |
| feitan…chrolo | w51–w57 |

Derive addresses from PKs; don’t guess truncated addresses.

---

## Pitfalls

1. **Console can click tiles but not always fill React inputs** — use proper type/click tools for forms  
2. **“OPEN POST” navigates away** — return to the WL URL; progress may persist  
3. **Deterministic puzzles** — same scramble after reload; reuse swap plan  
4. **PoW too fast** — backend “warming up” → dwell ~5s, retry with new challenge  
5. **Placeholder comment URLs** — many backends reject them; manual review will too  
6. **Proof URL already used** — mint a new reply for retries  
7. **Bare REST 429 mid-wave** — switch remaining accounts to Cloak  
8. **Cosmetic localStorage WL** — no server credit; don’t claim “submitted”  
9. **X soft-fail** — HTTP 200 with empty create_tweet ≠ success  
10. **Never invent** quote/tweet IDs for forms or reports  

---

## Compare: transport choices

| Situation | Prefer |
|---|---|
| Public Supabase / GForm / open JSON API | Bare REST |
| Cookie session + multi-akun burst | Cloak in-page fetch |
| Puzzle UI unlock only | Browser solve + then API if possible |
| X follow/RT/quote | `x.py` (+ follow 403 workaround) |
| Live OpenSea SeaDrop list check | SIWE + GraphQL eligibility skill |

---

## FAQ

**Q: Can I submit 8 wallets without 8 browser profiles?**  
A: Often yes if the backend is a public/custom API — one reverse, N posts (Cloak for throttles).

**Q: Site says already submitted after wallet #1?**  
A: UI lock is often `localStorage`. API path may still accept other wallets.

**Q: Do I need real X actions if the API doesn’t verify?**  
A: If the project says manual review / comment links checked, yes. Don’t farm fake proofs.

**Q: Puzzle solved but SUBMIT disabled?**  
A: Progress steps (open post / comment) may be client-gated — complete required steps or use the real API payload the client would send.

**Q: Relation to mint skills?**  
A: This gets you **on the list**. Minting later is [hermes-nft-skills](https://github.com/didin-lab/hermes-nft-skills).

---

## Security & ethics

- Use only accounts/wallets you control  
- Don’t publish tokens, cookies, or private keys  
- Respect site ToS and platform rules  
- Prefer low volume + delays over burst spam  

---

## Companion (Indonesian, shorter)

A simpler Indonesian version lives on Notion (portfolio hub):

→ https://app.notion.com/p/Web-WL-Gate-di-Hermes-ringkas-3a6dddfb96cd818eb376ecf14d646b0f

---

## Related portfolio guides

- [hermes-x-multi-account](https://github.com/didin-lab/hermes-x-multi-account)  
- [x-auto-reply-safe](https://github.com/didin-lab/x-auto-reply-safe)  
- [opensea-eligibility-check](https://github.com/didin-lab/opensea-eligibility-check)  
- [hermes-nft-skills](https://github.com/didin-lab/hermes-nft-skills)  

---

## License

MIT (documentation). Third-party sites, Discord/X/OpenSea, and projects have their own terms.
