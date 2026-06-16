# Electronic Voting System — Code Study

> **Note on authorship:** This is a reference implementation I am studying to
> learn modern web security patterns (JWT authentication, bcrypt password
> hashing, tamper-evident hash chains, rate limiting, and helmet security
> headers). The code was generated with the help of an AI coding assistant
> (Cowork) as a learning resource. **It is not a project I built myself from
> scratch.** I am keeping it here, in the open, as a study artifact and as a
> goal to eventually rebuild myself once I have internalised the patterns it
> demonstrates.
>
> For a project I genuinely built end-to-end, see my **Supermarket POS**
> system at <https://github.com/SirTaiwo/supermarket-pos>.

---

## What the system does

A small, security-aware electronic voting (e-voting) web application set in a
Nigerian elections context. It demonstrates:

- voter registration & authentication
- an administrator dashboard for managing elections
- live, public results with auto-refreshing tallies
- vote receipts the voter can verify independently
- a tamper-evident hash chain over the recorded votes
- an audit log of every meaningful action

**Stack:** Node.js · Express · SQLite (`better-sqlite3`) · JWT · bcrypt ·
vanilla JS + Chart.js

---

## Why I am studying this

My current portfolio project (the Supermarket POS) uses *session-based*
authentication via Passport. To grow as a developer, I want to also
understand:

| Concept | Why it matters |
|---|---|
| **JWT (token) authentication** | the dominant pattern for modern APIs and SPAs |
| **bcrypt** | password hashing done correctly, with a configurable cost |
| **Hash chains** | the core idea behind tamper-evident logs and blockchains |
| **Salted identity hashes** | how to record "this voter voted" without storing "who they voted for" |
| **Rate limiting & helmet headers** | basic production hardening |
| **Race-condition-safe writes** | UNIQUE constraints plus transactions |

Walking through this codebase line by line, with a teacher, is how I am
learning these patterns. Once I have understood them well enough to defend
every line in an interview, I plan to rebuild the system **on my own**,
honestly, as a project I can put on my CV.

---

## How to run it (for the curious)

You need **Node.js 18+** installed.

```bash
npm install
cp .env.example .env       # Windows: copy .env.example .env
npm run seed
npm start
```

Then open <http://localhost:3000>.

### Demo accounts
| Role  | Login | Password |
|-------|-------|----------|
| Admin | `admin` | `admin123` |
| Voter | `VIN0000001` … `VIN0000005` | `voter123` |

A sample **Presidential Election 2027 (Demo)** is created and opened with four
candidates.

To wipe and reseed at any time: `npm run reset`.

---

## Security features in this reference implementation

(These are the things I am studying, not things I have personally designed.)

- **Password hashing** — bcrypt (cost 12); plaintext passwords are never stored.
- **Token sessions** — JWT, signed server-side, expire after 2 hours;
  role-based access (voter vs admin) enforced on every protected route.
- **One vote per voter per election** — enforced by a database `UNIQUE`
  constraint *and* re-checked inside a transaction, so it cannot be bypassed
  by race conditions.
- **Ballot secrecy** — a vote stores a salted SHA-256 hash of the voter's
  identity, not the voter id, so a recorded vote cannot be trivially traced
  back to a person.
- **Vote receipt** — each voter gets a unique receipt to independently verify
  their vote was counted, without exposing their choice.
- **Tamper-evident hash chain** — every vote is linked to the previous one
  (`prev_hash → vote_hash`, SHA-256), like a lightweight blockchain. The
  admin *Integrity Check* recomputes the whole chain and flags the exact
  vote where any tampering occurred.
- **Audit log** — registrations, logins, votes, and all admin actions are
  recorded.
- **Hardening** — `helmet` security headers, rate limiting on auth and voting
  endpoints, request size limits, and input validation.

---

## What this is **not**

- Not a production-ready voting system. A real national system would need
  HTTPS/TLS, biometric voter verification (e.g. INEC's BVAS), a hardened
  database (PostgreSQL), independent end-to-end verifiability with published
  cryptographic proofs, professional security auditing, and a legal /
  regulatory framework. The hash-chain and receipt mechanisms here only
  *illustrate* the underlying ideas.
- **Not a project I built from scratch.** See the note at the top.

---

## My genuine portfolio

If you are reviewing my work, please go to:

- **Supermarket POS** — <https://github.com/SirTaiwo/supermarket-pos>
  (Live demo: <https://supermarket-pos-1lpe.onrender.com>)

That is the project I built, debugged, deployed and can fully defend.

— *Ogungbola Taiwo Moses (K. Ogungbola)*
